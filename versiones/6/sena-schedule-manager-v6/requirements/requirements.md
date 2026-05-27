```yaml
requirements:
  prd_document: |
    # SENA Schedule Manager V6 PRD

    The product is an MVP for managing SENA training schedules with Go,
    React, MongoDB, Docker and Docker Compose. The system supports classroom,
    student group, instructor, schedule and observation management. The target
    user is a schedule coordinator who needs a simple operational tool to
    create catalog records, create schedules with validated time ranges, and
    register observations for follow-up.

    This MVP intentionally reduces functional scope, not technical quality.
    The generated workspace must keep database, backend and frontend as
    separate components. Database schema/init/index/seed ownership belongs in
    the database component. Backend domain/core must stay independent from
    MongoDB and transport details; persistence documents and mappers belong in
    repository/infrastructure. Frontend must use a service/API layer and must
    not become a monolithic App component.

    Technical identifiers, source code, API contracts, diagrams, container
    names and database objects must use English. Spanish is allowed only for
    user-facing copy and functional documentation. The run is a fire test of
    the Agentic SDLC OS, so QA, AppSec and release evidence must be generated
    as real artifacts before completion.
  functional_requirements:
    - Create and list classroom records with code, name, location and capacity.
    - Create and list student group records with code, program name and learner count.
    - Create and list instructor records with staff or contractor type.
    - Create and list schedule records linked to classroom, student group and instructor.
    - Validate schedule day, start time and end time before persistence.
    - Create and list observations linked to a schedule.
    - Expose backend health and CRUD API endpoints for MVP entities.
    - Provide frontend screens to list and create MVP entities through an API client.
  non_functional_requirements:
    architecture: "database, backend and frontend are independent workspace components with their own container boundaries."
    maintainability: "no source file exceeds 300 lines; no monolithic main, App, page, repository, handler or CSS file."
    language: "technical identifiers, source code, API contracts and database objects use English; UI copy may be Spanish."
    data: "MongoDB collections and attributes use English singular lower_snake_case naming where applicable."
    domain_boundary: "domain/core code contains no persistence or transport metadata; persistence models and mappers live in repository/infrastructure."
    testing: "backend and frontend include real tests with executable commands."
    security: "backend sanitizes internal errors, restricts CORS, declares timeouts and avoids client-side secret storage."
    release: "Docker Compose, release readiness, deployment plan and rollback plan are produced before completion."
  user_stories:
    - id: US-001
      as: "schedule coordinator"
      i_want: "to create and list classrooms"
      so_that: "schedules can be assigned to physical environments"
    - id: US-002
      as: "schedule coordinator"
      i_want: "to create and list student groups"
      so_that: "training groups can be scheduled accurately"
    - id: US-003
      as: "schedule coordinator"
      i_want: "to create and list instructors with staff or contractor type"
      so_that: "teaching assignments distinguish employment relationship"
    - id: US-004
      as: "schedule coordinator"
      i_want: "to create schedules with classroom, group, instructor, day and time range"
      so_that: "training sessions can be planned consistently"
    - id: US-005
      as: "schedule coordinator"
      i_want: "to attach observations to schedules"
      so_that: "follow-up notes remain connected to the schedule record"
    - id: US-006
      as: "schedule coordinator"
      i_want: "to see validation errors for missing fields or invalid time ranges"
      so_that: "bad schedules are not persisted"
  acceptance_criteria:
    - "Docker Compose can start database, backend and frontend as distinct services."
    - "MongoDB init, indexes and seed artifacts live under database, not backend."
    - "Backend exposes health and CRUD endpoints for classroom, student_group, instructor, schedule and observation."
    - "Backend domain/core files contain no bson, gorm, db, sql, ORM/ODM annotations or DB driver imports."
    - "Frontend lists and creates MVP records via an API client with timeout handling."
    - "Backend and frontend include real tests and declared verification commands."
    - "A09 produces QA evidence with test commands and defect classification."
    - "A12 produces AppSec evidence with OWASP/threat/control/risk coverage and runtime evidence."
    - "A11 produces release readiness, deployment and rollback evidence."
  scope_boundaries:
    in_scope:
      - "classroom management"
      - "student group management"
      - "instructor management"
      - "schedule management"
      - "schedule observations"
      - "Docker Compose local runtime"
      - "QA, AppSec and release evidence"
    out_of_scope:
      - "authentication and role management"
      - "calendar drag-and-drop"
      - "massive reporting"
      - "import/export"
      - "production cloud deployment"
  stakeholders:
    - "schedule coordinator: primary operational user"
    - "SENA academic coordination: business owner"
    - "technical reviewer: validates architecture, QA, security and release readiness"
  quality_score: 0.91
```
