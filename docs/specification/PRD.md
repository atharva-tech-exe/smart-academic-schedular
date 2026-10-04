# PRD v3 — Smart Academic Scheduling Platform

## Product

School-first, multi-tenant SaaS for automatic academic timetable generation. Colleges are a later expansion. Core promise: **minimum manual effort, maximum useful output**.

## School MVP

- Organization, authentication, memberships and RBAC
- One active Institution per Organization
- 30-day trial entitlement
- School/institution and academic year/term
- Standard repeating weekly ScheduleCycle
- Classes, sections and learner groups
- Subjects
- Teachers, availability and workload policies
- Teaching requirements
- Rooms, room features and room availability
- Flexible bell schedules, ScheduleDays and explicit TimeSlots
- CSV/XLSX import with validation and atomic commit
- Preflight validation
- CP-SAT generation
- Independent validation
- Async generation jobs
- Review, Move/Swap, Lock
- Draft and immutable published versions
- PDF/XLSX/CSV export
- Audit log

## Core Flow

Landing → Organization Login → School Setup → Academic Context →
Classes/Sections → Subjects → Teachers → Teaching Requirements → Rooms →
Bell Schedule → Validate → Generate → Review → Move/Swap/Lock → Draft →
Publish → Export.

Use a flexible setup checklist, not a rigid one-way wizard.

## Core Scheduling Terminology

### ScheduleCycle

For MVP, `ScheduleCycle` is the recurring pattern over which TeachingRequirement frequencies are fulfilled. The MVP uses one standard repeating week. A/B rotating weeks and arbitrary multi-week rotation are post-MVP.

### TeachingRequirement

TeachingRequirement is the canonical scheduling requirement. It defines:

- subject
- one or more learner groups
- one or more teachers where team teaching is required
- `sessions_per_cycle`
- `slots_per_session`
- minimum/maximum occurrences per ScheduleDay where configured
- allowed/forbidden TimeSlots
- room requirement/policy
- named soft preferences

`session_length` is not used as an ambiguous synonym for `slots_per_session`.

An **occurrence** is one scheduled delivery of a TeachingRequirement. If:

- `sessions_per_cycle = 2`
- `slots_per_session = 2`

the requirement needs two occurrences, each occupying two consecutive compatible TimeSlots.

For MVP, one multi-slot occurrence must:
- occupy consecutive TimeSlots
- remain on the same ScheduleDay
- contain no break/non-teaching slot
- use a preflight-approved compatible slot group

Different TimeSlot durations do not automatically make adjacent slots compatible. Compatibility is determined by explicit domain/preflight rules.

## Learner Groups

Every Section has exactly one primary LearnerGroup representing all learners in that section.

A Section may have additional scheduling subgroups for activities such as split practical/computer groups.

MVP rules:
- arbitrary overlapping learner groups are not allowed unless an explicit relationship defines their conflict semantics
- a subgroup partition identifies sibling groups that may schedule simultaneously when they are disjoint
- a whole-section group conflicts with each subgroup belonging to its partition
- sibling subgroups from the same disjoint partition do not conflict with each other
- learner-group collision relationships are part of the canonical SchedulingProblem

Full student-level membership and arbitrary overlapping cohorts are post-MVP.

## Time

Use explicit TimeSlots rather than only period count.

Model:
`Calendar → BellSchedule → ScheduleDay → TimeSlot`

A ScheduleDay is an explicit persisted schedule-day definition. Different ScheduleDays may have different TimeSlot structures.

Support:
- different day schedules
- variable durations
- breaks
- assemblies/activities
- fixed non-teaching slots
- double/multi-slot sessions

A multi-slot session can use only a preflight-approved contiguous compatible TimeSlot group.

## Rooms

Rooms have:
- capacity
- type/features
- explicit availability
- optional default classroom relationship
- room requirements

Room allocation policy belongs to the TeachingRequirement's room requirement and supports:
- `fixed`
- `preferred`
- `flexible`

A Section's normal classroom may be a default/preference, but a TeachingRequirement may override it for labs/special rooms.

## Hard Constraints

At minimum:

- learner-group collision rules
- teacher overlap
- room overlap
- teacher availability
- room availability
- required occurrence count
- `slots_per_session`
- compatible consecutive slot placement
- capacity
- required room features
- fixed activities/non-teaching slots
- timetable locks
- configured teacher workload limits
- fixed-room requirements

Published schedules must have zero hard violations.

## Initial Hard Workload Policy

The MVP vocabulary is intentionally small:
- unavailable TimeSlots
- maximum occurrences per ScheduleDay
- maximum consecutive slots
- optional maximum occurrences per ScheduleCycle

No arbitrary workload JSON is required for MVP.

## Soft Constraints

Named MVP objectives:

1. minimize teacher gaps
2. minimize learner-group gaps
3. spread repeated subject occurrences across ScheduleDays where possible
4. avoid undesirable subject adjacency where configured
5. honor room preferences
6. honor teacher preferences if enabled
7. minimize unnecessary teacher consecutive load where configured

Each objective has an explicit penalty unit and weight. There is no opaque single quality score.

## Objective Configuration

`ConstraintConfiguration` stores:
- enabled objectives
- default/configured weights
- objective version
- applicable scope

The configuration is captured by `GenerationInputRevision`, so a generation can always be reproduced with the same objective configuration.

MVP institutions may configure weights only for the supported objective set. Adding new objective types is a specification/code change, not arbitrary user-defined solver expressions.

## Scheduler

CP-SAT is authoritative. Greedy is optional for hints. Branch and Bound is not primary. Local Search is not MVP. Never publish an unvalidated fallback.

Pipeline:

Input Revision → Preflight → Canonical SchedulingProblem → Constraints →
Optional Hints → CP-SAT → Independent Validator → Metrics/Diagnostics →
Candidate Draft TimetableVersion.

## Async Generation

API → GenerationJob → Worker → Solver → Validator → Result.

A generation request is tied to an exact `GenerationInputRevision` and an idempotency key/request identity. Repeating the same client request for the same input revision must not create duplicate expensive solves.

Job states:
`queued → preflight → solving → validating → completed/failed/cancelled`

A successful GenerationJob produces exactly one candidate draft TimetableVersion referencing exactly one GenerationInputRevision. Failed/cancelled jobs produce no publishable timetable version.

## Versioning

Published versions are immutable.

Editing a draft uses server-side commands and optimistic revision checks. Publishing creates the next immutable version.

Move/Swap receives an intended operation, not a complete timetable snapshot. The server checks authorization, current draft revision, locks and hard constraints before atomically applying the mutation.

## Import

Upload → Parse → Map → Validate → Preview → Errors/Warnings → Confirm → Atomic Import.

Import uses deterministic entity identity/upsert rules. Stable external/internal identifiers are preferred over names.

Preview is bound to an import validation revision. If relevant academic data changes before confirmation, the stale preview is rejected and must be revalidated.

## Organization / Institution / Membership

MVP:
- one Organization has exactly one active Institution
- schema may remain extensible for future one-to-many support
- OrganizationMembership is unique on `(organization_id, user_id)`
- membership statuses: `invited`, `active`, `suspended`
- authorization requires active membership

## Trial / Entitlements

The first eligible Organization activation creates one non-renewable 30-day trial.

Trial expiry preserves customer data but blocks premium mutation/generation capabilities according to EntitlementService. Explicitly allowed read/export capabilities remain available.

Plan names are never inspected directly inside business endpoints.

Phase 2 contains only the plan/entitlement representation needed for the trial. Payment subscriptions and billing events remain Phase 9.

## MVP exclusions

Exams, seating, invigilation, college workflow, alternatives, partial regeneration, local search, advanced analytics, AI import mapping, existing timetable migration, notifications, multi-campus, what-if, complex drag/drop and visual version comparison.

## Future

Phase 2+: examinations and advanced scheduling.
Phase 3+: college scheduling, electives, practical groups, cross-department constraints, smart import and multi-campus.


## v3.1 Academic Context and Revision Invariants

The MVP scheduling context is scoped through:

`Organization → Institution → AcademicYear → AcademicTerm → ScheduleCycle`

and:

`AcademicTerm → Calendar → BellSchedule → ScheduleDay → TimeSlot`.

A TeachingRequirement, GenerationInputRevision, and TimetableVersion each belong to one AcademicTerm/ScheduleCycle context.

LearnerGroup partitions are deterministic: sibling subgroups within the same explicitly disjoint partition may run simultaneously; groups from different partitions of the same Section conflict by default unless an explicit disjoint relationship exists; the whole-section group conflicts with all subgroups.

Each schedulable AcademicTerm has an authoritative AcademicDataRevision. Scheduling-relevant academic changes increment it atomically. Import previews bind to this revision, and stale previews cannot be confirmed. GenerationInputRevision captures an immutable solver-input snapshot derived from one exact AcademicDataRevision.
