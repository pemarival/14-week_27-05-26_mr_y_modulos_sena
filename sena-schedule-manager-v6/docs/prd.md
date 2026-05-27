# SENA Schedule Manager V6 - Fire Test PRD

## Objective

Build an MVP for SENA schedule management using:

- Backend: Go
- Frontend: React
- Database: MongoDB
- Runtime: Docker and Docker Compose

This is a fire test for the agentic ecosystem. The product must be functional
enough to exercise the full SDLC runtime, but MVP means reduced functional
scope, not reduced architecture, testing, security, naming or release quality.

## Functional Scope

The MVP must support:

- Classroom management.
- Schedule management.
- Student group management.
- Instructor management for staff and contractor instructors.
- Observations linked to schedules.

## Technical Rules

- User-facing UI copy and user documentation may be Spanish.
- Source code, diagrams, API contracts, container names, database objects,
  database attributes and technical identifiers must be English.
- Database collections and attributes must be English, singular and lower
  snake case where applicable.
- The workspace must separate `database/`, `backend/` and `frontend/` as
  independent components.
- The database must not be embedded inside the backend.
- Each component must have its own container boundary.
- Domain/core code must not contain persistence or transport metadata such as
  `bson`, `gorm`, `db`, `sql`, JPA, Mongoose, Prisma, TypeORM or DB driver
  imports.
- Persistence models/documents/records must live in repository/infrastructure
  with explicit mappers to/from domain.
- No source file may exceed 300 lines.
- No monolithic `main.go`, `App.jsx`, page, repository, handler or CSS file.
- Backend must include real tests.
- Frontend must include real tests and reproducible build metadata.
- QA, AppSec and release evidence must be real, not narrative-only.

## MVP User Stories

1. As a coordinator, I can create and list classrooms so schedules can be
   assigned to a physical environment.
2. As a coordinator, I can create and list student groups so schedules can be
   associated with a training group.
3. As a coordinator, I can create and list instructors, including staff and
   contractor type.
4. As a coordinator, I can create schedules by selecting classroom, group,
   instructor, day, start time and end time.
5. As a coordinator, I can add observations to a schedule for follow-up.
6. As a coordinator, I can see validation errors when required fields are
   missing or schedule time ranges are invalid.

## Acceptance Criteria

- The app can be started locally with Docker Compose.
- The backend exposes health and CRUD endpoints for the MVP entities.
- The frontend can list and create MVP entities through an API client.
- MongoDB initialization, indexes and seed data live in `database/`.
- Backend tests can be run with the declared stack command.
- Frontend tests/build can be run with the declared stack command.
- A09 produces QA evidence.
- A12 produces AppSec evidence.
- A11 produces release readiness, deployment and rollback evidence.

## Out Of Scope

- Authentication and role management.
- Calendar drag-and-drop.
- Massive reporting.
- Import/export.
- Production cloud deployment.
