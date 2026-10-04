# Tasks v3

## Phase 0 — Specification Freeze

- [x] Freeze school MVP
- [x] Confirm exams/college are post-MVP
- [x] Finalize Organization/Institution/Membership
- [x] Finalize Academic Year/Term/Schedule Cycle
- [x] Finalize Learner Group
- [x] Finalize Teaching Requirement
- [x] Finalize Bell Schedule/TimeSlot
- [x] Finalize Room/Feature model
- [x] Finalize Timetable Version
- [x] Freeze hard constraints
- [x] Freeze initial soft constraints
- [x] Define objective configuration
- [x] Define infeasibility diagnostics
- [x] Define permission matrix
- [x] Define tenancy/auth invariants
- [x] Define trial/entitlement invariants
- [x] Define generation idempotency
- [x] Define import identity/upsert rules
- [x] Define stale import-preview handling
- [x] Define draft optimistic concurrency
- [x] Define Move/Swap API command semantics
- [x] Create realistic test-school dataset requirements
- [x] Complete final Phase 0 review


## Phase 0.1 — Final v3.1 Clarification Gate

Complete these architectural freezes before Phase 1:
1. Freeze Organization → Institution → AcademicYear → AcademicTerm → ScheduleCycle ownership.
2. Freeze AcademicTerm → Calendar → BellSchedule → ScheduleDay → TimeSlot ownership.
3. Freeze TeachingRequirement and TimetableVersion scope to one AcademicTerm/ScheduleCycle.
4. Freeze cross-partition LearnerGroup collision semantics.
5. Freeze the AcademicDataRevision lifecycle and its relationship to Import Preview and GenerationInputRevision.

These are specification gates, not implementation work. Once all five are documented consistently, Phase 0 is approved and Phase 1 may begin.

## Phase 1 — Foundation

- [ ] Git repository
- [ ] React/TypeScript/Vite
- [ ] FastAPI
- [ ] PostgreSQL
- [ ] SQLAlchemy
- [ ] Alembic
- [ ] Environment configuration
- [ ] Logging
- [ ] Tests
- [ ] Lint/format
- [ ] CI
- [ ] Local object-storage adapter
- [ ] README

## Phase 2 — Identity/Tenancy/Entitlements

- [ ] User
- [ ] Organization
- [ ] Institution
- [ ] Membership
- [ ] Role/Permission
- [ ] Registration
- [ ] Verification
- [ ] Login/logout
- [ ] Password reset
- [ ] Invitations
- [ ] Tenant context
- [ ] Cross-tenant tests
- [ ] Audit log
- [ ] Plan representation
- [ ] Trial entitlement
- [ ] Entitlement service
- [ ] Membership lifecycle enforcement

## Phase 3 — Calendar/Time

- [ ] Academic year
- [ ] Academic term
- [ ] Standard weekly ScheduleCycle
- [ ] Calendar
- [ ] Working days
- [ ] Bell schedule
- [ ] ScheduleDay
- [ ] Time slots
- [ ] Breaks
- [ ] Fixed activities
- [ ] Variable day schedules
- [ ] Double/multi-slot compatibility
- [ ] Time validation
- [ ] Compatible slot-group precomputation

## Phase 4 — School Academic Model

- [ ] Class/grade
- [ ] Section
- [ ] Primary learner group
- [ ] Learner-group partitions
- [ ] Learner strength
- [ ] Subject
- [ ] Teacher
- [ ] Teacher availability
- [ ] Teacher workload policy
- [ ] Room
- [ ] Room availability
- [ ] Room features
- [ ] Room feature assignment
- [ ] Fixed/preferred/flexible room mode
- [ ] Teaching requirement
- [ ] Requirement teachers
- [ ] Requirement learner groups
- [ ] Room requirements
- [ ] Explicit learner-group collision relationships

## Phase 5 — Import

- [ ] CSV
- [ ] XLSX
- [ ] Templates
- [ ] Column mapping
- [ ] Validation
- [ ] Stable identity matching
- [ ] Create/update/ambiguous actions
- [ ] Preview revision
- [ ] Errors/warnings
- [ ] Atomic import
- [ ] Stale-preview detection
- [ ] Import history
- [ ] Stored object metadata

## Phase 6 — Scheduler MVP

- [ ] SchedulingProblem
- [ ] SchedulingSolution
- [ ] Generation input revision
- [ ] Constraint configuration
- [ ] Preflight validator
- [ ] Compatible placement groups
- [ ] Learner-group conflict graph
- [ ] Hard constraints
- [ ] Soft objectives
- [ ] CP-SAT model
- [ ] Optional heuristic hints
- [ ] Time limit
- [ ] Cancellation
- [ ] Independent validator
- [ ] Metrics
- [ ] Diagnostics
- [ ] Infeasibility reporting

## Phase 7 — Async Generation/Timetable

- [ ] Generation job
- [ ] Generation idempotency
- [ ] Worker/queue
- [ ] Job status API
- [ ] Cancellation
- [ ] Concurrency limits
- [ ] Candidate draft timetable version
- [ ] Entries
- [ ] Draft revision
- [ ] Locks
- [ ] Class view
- [ ] Teacher view
- [ ] Room view
- [ ] Move command
- [ ] Swap command
- [ ] Optimistic concurrency handling
- [ ] Publish
- [ ] Immutable published version

## Phase 8 — Export

- [ ] PDF
- [ ] XLSX
- [ ] CSV
- [ ] Print
- [ ] Export jobs
- [ ] Exact version references
- [ ] Secure downloads

## Phase 9 — Billing

- [ ] Select payment provider
- [ ] Customer/subscription mapping
- [ ] Payment integration
- [ ] Verified webhooks
- [ ] Idempotent webhook handling
- [ ] Subscription state
- [ ] Renewal
- [ ] Cancellation
- [ ] Billing history
- [ ] Plan limits
- [ ] Upgrade UI

## Phase 10 — Pilot/Hardening

- [ ] Realistic school datasets
- [ ] Infeasible datasets
- [ ] Solver benchmarks
- [ ] Tenant isolation tests
- [ ] Import failure tests
- [ ] Concurrent generation tests
- [ ] Generation idempotency tests
- [ ] Version immutability tests
- [ ] Draft concurrency tests
- [ ] Export consistency tests
- [ ] Security review
- [ ] Backups
- [ ] Monitoring
- [ ] Deployment
- [ ] Pilot feedback

## Post-MVP — Examination

- [ ] Students
- [ ] Enrollment/registration
- [ ] Exam definition
- [ ] Exam sessions
- [ ] Exam scheduling
- [ ] Exam room configuration
- [ ] Seating
- [ ] Seating solver
- [ ] Invigilation
- [ ] Attendance sheets

## Post-MVP — Advanced Scheduling

- [ ] Multiple alternatives
- [ ] Partial regeneration
- [ ] Change-minimization
- [ ] Version comparison
- [ ] Advanced scoring
- [ ] What-if
- [ ] Existing timetable migration
- [ ] Smart Excel mapping
- [ ] Arbitrary rotating ScheduleCycles
- [ ] Arbitrary overlapping learner groups
- [ ] Student-level cohort constraints

## Post-MVP — College

- [ ] Departments
- [ ] Programs
- [ ] Semesters
- [ ] Divisions
- [ ] Batches
- [ ] Course offerings
- [ ] Electives
- [ ] Practical groups
- [ ] Cross-department scheduling
- [ ] Student/cohort overlap constraints

**Rule:** do not skip ahead because a later feature is more exciting.
