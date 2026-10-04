# Project Memory v3

## Product

Smart Academic Scheduling Platform. School-first B2B SaaS; college later.

Promise: **Minimum manual effort → maximum useful output.**

## MVP

Organization/auth/membership/RBAC, one active Institution per
Organization, 30-day trial entitlement, academic context, standard
weekly ScheduleCycle, classes/sections/learner groups, subjects,
teachers/availability/workload, teaching requirements, rooms/features/
availability, explicit ScheduleDays/TimeSlots, import/validation,
CP-SAT generation, independent validation, async generation,
Move/Swap/Lock, immutable published versions, exports and audit.

## Core model

Time + Resources + Learner Groups + Teaching Requirements +
Constraints + Scheduling Problem + Solution Version.

## TeachingRequirement

`sessions_per_cycle` is the number of lesson occurrences required during
the ScheduleCycle.

`slots_per_session` is the number of consecutive compatible TimeSlots
occupied by each occurrence.

Example: 2 occurrences × 2 slots = two separate double-slot lessons.

Each occurrence stays on one ScheduleDay and uses a preflight-approved
compatible contiguous slot group.

## LearnerGroup

Every Section has one primary whole-section LearnerGroup.

Additional subgroups belong to the Section and may be organized into
disjoint LearnerGroupPartitions.

Sibling groups in a partition may run simultaneously. The whole-section
group conflicts with its partition subgroups. Arbitrary overlaps are
not assumed in MVP.

## Time

`Calendar → BellSchedule → ScheduleDay → TimeSlot`

MVP ScheduleCycle is one standard repeating week.

Multi-slot compatibility is explicit and preflight-computed.

## Rooms

Room has capacity, features/type and persisted RoomAvailability.

TeachingRequirementRoomRequirement controls:
- fixed
- preferred
- flexible

## Solver

CP-SAT authoritative. Greedy optional hints. Branch and Bound not
primary. Local Search not MVP. Never publish an unvalidated fallback.

## Objectives

Named soft objectives with explicit penalty units and versioned weights:
teacher gaps, learner-group gaps, subject spread, subject adjacency,
room preference, teacher preference and unnecessary consecutive load.

## Generation

API → Generation Job → Worker → Solver → Validator → Result.

Generation is tied to exactly one GenerationInputRevision and an
idempotency key.

Successful job → exactly one candidate draft TimetableVersion.
Failed/cancelled job → no publishable version.

## Versioning

Published versions immutable.

Draft mutations use optimistic revision checks.

TimetableLock is draft/version scoped.

## Import

Upload → Parse → Map → Validate → Preview → Confirm → Atomic Import.

Stable identifiers preferred. Ambiguous identity matches are rejected.
Stale previews are rejected and must be revalidated.

## SaaS

First eligible Organization activation creates one non-renewable
30-day trial. EntitlementService centralizes access/limits. Trial
expiry preserves data and blocks premium capabilities according to
entitlements.

## Security

Tenant resolved from active membership. Never trust client organization
ID. Auth, recovery, invitations, RBAC, audit and cross-tenant tests are
required.

## API boundaries

Academic mutations, scheduler commands and timetable lifecycle commands
are separate service boundaries even inside one FastAPI deployment.

## Future examination

Academic Data → Exam Definition → Registration → Exam Scheduling → Room
Allocation → Seating → Invigilation.

## Future college

Map departments/programs/semesters/divisions/batches/electives/practical
groups into the generic scheduling model.

## Workflow

Chat = architecture/decisions. Work = repository implementation. Plan →
Inspect → Implement → Test → Review → Commit. One bounded task at a
time.


## v3.1 Frozen Context

Canonical academic ownership:
`Organization → Institution → AcademicYear → AcademicTerm → ScheduleCycle`

Canonical time ownership:
`AcademicTerm → Calendar → BellSchedule → ScheduleDay → TimeSlot`

TeachingRequirement, GenerationInputRevision, and TimetableVersion are scoped to one AcademicTerm/ScheduleCycle.

LearnerGroup collision rule: same-partition siblings are disjoint; cross-partition groups conflict by default unless explicitly marked disjoint; whole-section group conflicts with all subgroups.

Each schedulable AcademicTerm has an authoritative AcademicDataRevision. Scheduling-relevant academic changes increment it atomically. Import previews bind to it, and GenerationInputRevision is an immutable solver-input snapshot derived from exactly one revision.
