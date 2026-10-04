# Design v3

## Direction

Professional education administration SaaS: clean, reliable, simple,
information-dense, desktop-first for administrators, responsive for
teachers, accessible and print-friendly.

## MVP Sidebar

Dashboard

SETUP
- Institution
- Academic Context
- Classes & Sections
- Subjects
- Teachers
- Teaching Requirements
- Rooms
- Bell Schedule

SCHEDULING
- Generate
- Timetable
- Conflicts

REPORTS
- Exports

ADMIN
- Users & Roles
- Subscription
- Settings

No examination navigation in MVP.

## Dashboard

Show:
- setup progress
- data validation state
- current timetable/version
- hard conflicts
- unscheduled occurrences
- trial/entitlement state
- required actions

## Setup

Use a flexible checklist. Users can revisit steps and imports can
complete multiple steps.

## Academic Context

Show Academic Year, Academic Term and the standard weekly ScheduleCycle.
Make the recurring-week semantics visible enough that administrators do
not mistake ScheduleCycle for a version boundary.

## Period/Bell Schedule

Use the supplied Period Configuration/Break Configuration screenshot as
the visual reference.

Controls should support:
- ScheduleDay
- TimeSlot start/end
- duration
- ordinal
- teaching/break/fixed-activity/non-teaching type
- working days
- break name
- after-period placement
- edit/delete

Backend must support variable day schedules, double/multi-slot sessions,
assemblies, activities and non-teaching slots.

When a multi-slot session is requested, only preflight-approved
compatible consecutive TimeSlots should be offered.

## Learner Groups

Section setup shows:
- primary whole-section LearnerGroup
- optional scheduling subgroups
- subgroup partition where applicable

UI should make partition membership and simultaneous scheduling behavior
clear.

Do not expose arbitrary student-level group construction in MVP.

## Teaching Requirements

The form explicitly uses:
- Sessions per cycle
- Slots per session
- Minimum/maximum occurrences per ScheduleDay where needed
- Subject
- Learner group(s)
- Teacher(s)
- Allowed/forbidden slots
- Room requirement
- Fixed/preferred/flexible room policy
- Named preferences

Avoid the ambiguous label "session length" unless it is explicitly
explained as a slot count.

## Validation

Before generation show:
- blocking errors
- warnings
- required occurrence feasibility
- learner-group conflict issues
- teacher availability/workload issues
- room availability/capacity/features issues
- incompatible multi-slot requirements
- stale/import revision problems
- readiness

## Generation

Show real job states:
`queued → preflight → solving → validating → completed/failed/cancelled`

Never invent progress percentages.

If a duplicate idempotent request is submitted, show/reuse the existing
job instead of starting another solve.

## Infeasibility

Explain specific blocking conditions and provide actions to review:
- unavailable resources
- impossible slot groups
- room capacity/features
- learner-group collisions
- teacher workload/availability
- fixed locks/activities

## Timetable

Primary views:
- Class
- Teacher
- Room

Each entry shows:
- subject
- teacher
- room
- relevant learner group where useful

## Editing

MVP:
click entry → Move/Swap dialog → valid destinations → server validation
→ save draft.

Move/Swap submits an operation plus expected draft revision. The server
checks locks and hard constraints and increments the revision atomically.

Complex drag/drop later.

## Locking

Show lock state.

Locks are draft/version scoped. Locked entries cannot be moved or
changed by generation/edit commands unless an authorized explicit
unlock operation is performed.

## Versioning

Published V1 → Draft revision → Published V2.

Published versions are immutable.

If two administrators edit the same draft, stale revision requests are
rejected rather than silently overwriting changes.

## Import

Download Template → Upload → Column Mapping → Validation → Preview →
Confirm.

Preview displays identity actions:
- create
- update
- reject/ambiguous

Confirmation is allowed only for the current validation revision.

## Export

Class/Teacher/Room/Complete timetable → PDF/XLSX/CSV/Print.

Always export an exact timetable version.

## Accessibility

Keyboard navigation, visible focus, adequate contrast, labels, useful
errors and non-color-only statuses.

## Responsive

Admin setup is desktop-first. Teacher timetable views work on
tablet/mobile. Timetable grids may horizontally scroll.

## Permissions

Hide unavailable actions in the UI, but always enforce permissions on
the server. Publish, generate, edit and user-management controls should
reflect the permission matrix.


## v3.1 Context and Revision UX Rules

Academic setup must make the active Institution, AcademicYear, AcademicTerm, ScheduleCycle, Calendar, and BellSchedule context explicit to the user.

Import Preview must display the academic-data revision/context it validated against. If the underlying scheduling-relevant academic data changes before confirmation, the UI must require revalidation rather than silently applying the stale preview.

Timetable generation and timetable review operate within one explicit AcademicTerm/ScheduleCycle context. The UI must not imply that unrelated academic contexts can be combined into one generation.
