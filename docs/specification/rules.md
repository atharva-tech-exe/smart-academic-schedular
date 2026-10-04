# Rules v3

## Product

1. School MVP first.
2. Minimize manual effort, not required information.
3. Reuse existing data.
4. Published schedules must have zero hard violations.
5. Never silently drop requirements.
6. Published versions are immutable.
7. Do not implement post-MVP examination/college features in the MVP.

## Domain

1. TeachingRequirement is the canonical scheduling requirement.
2. Do not model scheduling only as Teacher + Subject + Class.
3. Use learner groups and explicit TimeSlots.
4. `sessions_per_cycle` counts lesson occurrences.
5. `slots_per_session` counts compatible consecutive TimeSlots occupied by one occurrence.
6. A multi-slot occurrence cannot cross a break, fixed activity or ScheduleDay boundary.
7. Learner Groups have explicit collision semantics; arbitrary overlaps are not assumed.
8. Every Section has one primary whole-section LearnerGroup.
9. LearnerGroupPartition represents disjoint scheduling subgroups.
10. Support room capacity/features and fixed/preferred/flexible allocation.
11. Room availability is explicit and persisted.
12. Separate hard and soft constraints.

## Time

1. MVP ScheduleCycle is one standard repeating week.
2. ScheduleDay is explicit and persisted under BellSchedule.
3. TimeSlot type determines whether it can host teaching.
4. Multi-slot placement uses only preflight-approved compatible slot groups.
5. CP-SAT must not infer multi-slot compatibility from ordinal numbers alone.

## Solver

1. CP-SAT is authoritative.
2. Greedy is optional and may provide hints.
3. Branch and Bound is not primary.
4. Local Search is not MVP.
5. Never publish an unvalidated fallback.
6. Independent validation is mandatory.
7. Scheduler code stays independent from web/database code.
8. Use deterministic test datasets.
9. Record solver metadata.
10. Use async generation jobs.
11. A successful generation job produces one candidate draft version.
12. Failed/cancelled jobs produce no publishable version.
13. Generation requests are idempotent for the same organization/input revision/request identity.

## Hard constraints

1. No teacher overlap.
2. No learner-group conflict according to explicit learner-group relationships.
3. No room overlap.
4. Teacher availability.
5. Room availability.
6. Required occurrence count.
7. `slots_per_session`.
8. Compatible consecutive slot placement.
9. Capacity.
10. Required room features.
11. Fixed activities/non-teaching slots.
12. Fixed-room requirements.
13. Timetable locks.
14. Configured teacher workload limits.

## Initial teacher workload vocabulary

Only these are MVP hard policy controls:
- unavailable TimeSlots
- max occurrences per ScheduleDay
- max consecutive teaching slots
- optional max occurrences per ScheduleCycle

Do not create an arbitrary policy JSON container for MVP.

## Soft objectives

Named objectives only:
- teacher gaps
- learner-group gaps
- subject spread
- subject adjacency avoidance
- room preference
- teacher preference
- unnecessary consecutive teaching load

Each objective has a stable key, penalty unit, enabled state and
versioned weight. No opaque overall quality score.

## Database/Security

1. PostgreSQL + SQLAlchemy + Alembic.
2. Tenant data is membership-scoped.
3. Never trust client organization IDs.
4. Test cross-tenant isolation.
5. Avoid destructive deletion of referenced academic records.
6. Published versions are immutable.
7. Generation inputs are revisioned.
8. Never store payment card details.
9. Membership uniqueness is `(organization_id, user_id)`.
10. Only active memberships authorize tenant actions.
11. Draft timetable mutations use optimistic revision checks.

## RBAC

1. Authorization uses explicit permissions, not scattered role-name checks.
2. Initial permissions are:
   - timetable.view
   - academic.manage
   - import
   - export
   - schedule.generate
   - schedule.edit
   - schedule.lock
   - schedule.publish
   - users.manage
   - billing.manage
3. Organization Admin receives all MVP permissions.
4. Academic Scheduler receives timetable/academic/import/export/scheduling permissions but not billing.
5. Teacher/Staff Viewer is read-only for authorized timetable views/exports.
6. Additional grants may be represented through permissions without changing the core model.

## Import

1. Upload → Parse → Map → Validate → Preview → Confirm → Atomic Import.
2. Stable identifiers are preferred.
3. Ambiguous identity matches fail validation rather than silently duplicating or updating.
4. Confirmation must reference the validated import revision.
5. Relevant academic-data changes invalidate stale previews.
6. Import commits atomically.

## API

1. Validate inputs.
2. Enforce authorization server-side.
3. Paginate large lists.
4. Use jobs for long operations.
5. Tie exports to exact timetable versions.
6. Separate academic mutations, scheduler commands and timetable lifecycle commands.
7. Move/Swap accepts operations, not authoritative full-timetable snapshots.
8. Move/Swap checks draft revision, locks and hard constraints before atomic commit.

## UI

1. Use guided checklists.
2. Clear required/optional states.
3. Useful empty states.
4. Server-validated Move/Swap dialogs.
5. Show real generation job states.
6. Complex drag/drop is later.
7. Never invent solver progress percentages.

## SaaS

1. Trial starts server-side.
2. First eligible organization receives one non-renewable 30-day trial.
3. Entitlements are centralized.
4. Trial expiry does not delete academic data.
5. Payment webhooks must be verified and idempotent in Phase 9.
6. Subscription expiry must not delete academic data.

## Testing

Test:
- teacher conflicts
- learner-group conflicts and partition behavior
- room conflicts
- availability
- capacity
- features
- occurrence count
- slots per session
- compatible multi-slot placements
- fixed activities
- locks
- workload
- infeasible cases
- generation idempotency
- draft optimistic concurrency
- stale import previews
- import identity ambiguity
- cross-tenant negative cases
- published-version immutability

## AI

Do not use an LLM as the deterministic scheduler. AI may later assist
with data mapping, cleanup, explanations and support.

## AI coding workflow

Plan → Inspect → Implement → Test → Review → Commit. One bounded task at
a time. Never claim tests passed unless actually run.


## v3.1 Frozen Invariants

### Academic Context
- Organization → Institution → AcademicYear → AcademicTerm → ScheduleCycle is the canonical ownership chain.
- AcademicTerm → Calendar → BellSchedule → ScheduleDay → TimeSlot is the canonical time chain.
- MVP has one active Institution per Organization.
- TeachingRequirement, GenerationInputRevision, and TimetableVersion are scoped to one AcademicTerm/ScheduleCycle.

### LearnerGroup Collisions
- Same-partition sibling subgroups are disjoint.
- Cross-partition groups conflict by default unless an explicit disjoint relationship exists.
- Whole-section groups conflict with all subgroups of the Section.
- Student-level membership is out of MVP.

### Academic Data Revision
- Each schedulable AcademicTerm has an authoritative AcademicDataRevision.
- Scheduling-relevant mutations increment the revision atomically.
- Import Preview records the revision and confirmation requires it to remain unchanged.
- GenerationInputRevision is an immutable solver-input snapshot derived from exactly one academic-data revision.
