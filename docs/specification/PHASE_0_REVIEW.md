# Specification v3.1 — Final Phase 0 Clarification Review

## Scope

Specification v3.1 is a narrow clarification pass over Specification v3. It resolves the three High issues identified by the final implementation-readiness review. It does not redesign the architecture.

## H1 — Academic-context ownership relationships

**Status: RESOLVED**

Canonical ownership:
`Organization → Institution → AcademicYear → AcademicTerm → ScheduleCycle`

Canonical time chain:
`AcademicTerm → Calendar → BellSchedule → ScheduleDay → TimeSlot`

TeachingRequirement, GenerationInputRevision, and TimetableVersion are scoped to exactly one AcademicTerm/ScheduleCycle.

MVP uses a standard repeating weekly ScheduleCycle; rotating A/B weeks remain out of MVP.

## H2 — Cross-partition LearnerGroup collision semantics

**Status: RESOLVED**

- One primary whole-section LearnerGroup per Section.
- A Section may contain multiple explicitly disjoint LearnerGroupPartitions.
- Sibling subgroups in the same partition are mutually disjoint and may run simultaneously.
- Groups from different partitions conflict by default.
- An explicit supported disjoint relationship may prove cross-partition groups disjoint.
- Whole-section groups conflict with all subgroups.
- Arbitrary student-level overlap is out of MVP.
- Preflight validation rejects contradictory or ambiguous relationships.

The canonical SchedulingProblem therefore receives deterministic learner-group conflict relationships.

## H3 — Academic-data revision lifecycle

**Status: RESOLVED**

Each schedulable AcademicTerm maintains an authoritative AcademicDataRevision.

Scheduling-relevant academic changes increment it atomically. Import Preview records the revision and confirmation requires the same revision. GenerationInputRevision is an immutable complete solver-input snapshot derived from exactly one AcademicDataRevision and records that source revision.

## Final readiness gate

Critical: 0
High: 0
Medium: 0
Low: 0

The remaining specification details are implementation-phase clarifications and do not require database, domain-model, tenancy, or CP-SAT architectural redesign before Phase 1.

## Final Decision

READY FOR IMPLEMENTATION

PHASE 0 APPROVED

## First implementation task

Phase 1 — Git repository / project foundation, as defined in tasks.md.

Specification v3.1 is now the implementation baseline. Specification v3 remains unchanged.
