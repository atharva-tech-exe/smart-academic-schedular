# Smart Academic Scheduler — Specification v3

## Status

**PHASE 0 APPROVED**

Critical: 0  
High: 0  
Medium: 0  
Low: 0

## Package

This package is Specification v3 and is intentionally separate from the
Specification v2 package. The v2 baseline must remain unchanged.

Files:
- PRD.md
- architecture.md
- rules.md
- design.md
- tasks.md
- memory.md
- PHASE_0_REVIEW.md

## What changed

Specification v3 resolves all 23 findings from the final Phase 0 review,
with special attention to:
- TeachingRequirement occurrence/slot semantics
- LearnerGroup partition and collision semantics
- ScheduleCycle/ScheduleDay
- TimeSlot compatibility
- Room availability and room policy
- workload and objective configuration
- RBAC/trial invariants
- generation idempotency and version relationships
- draft concurrency and locks
- deterministic import identity/upsert behavior
- API command boundaries

## Implementation rule

Do not redesign the architecture while implementing Phase 1 unless a
new requirement is explicitly approved. Follow `tasks.md` in order.

## First implementation task

**Git repository**


## Specification v3.1

This package is a narrow clarification pass over Specification v3. It does not redesign the architecture.

It freezes:
- academic context ownership and scheduling scope,
- cross-partition LearnerGroup collision semantics,
- the AcademicDataRevision lifecycle used by import preview validation and GenerationInputRevision.

The original Specification v3 package remains unchanged.
