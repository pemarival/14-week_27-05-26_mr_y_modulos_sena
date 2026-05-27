```yaml
requirements:
  prd: |
    # Product Requirements Document
    ## Project
    SENA Schedule Manager

    ## Product Goal
    Build a production-shaped MVP for managing SENA academic schedules. MVP means reduced functional scope, not reduced engineering quality.
    The product must support scheduling rooms, training groups, instructors, schedule blocks, and observations. The user-facing UI and user documentation must be in Spanish. All technical code, folder names, database object names, entity names, attributes, diagrams, API contracts, tests, comments, and developer documentation must be in English.

    ## Mandatory Stack
    - Backend: Go
    - Frontend: React
    - Database: MongoDB
    - Runtime: Docker and Docker Compose

    ## Component Boundaries
    - database/ – MongoDB init, seed, indexes, validation, Dockerfile.
    - backend/ – Go API only.
    - frontend/ – React UI only.
    - Root – docker-compose.yml, .env.example, .gitignore, cross-component docs.
    Each component has its own container.

    ## Functional Scope
    - Rooms: CRUD + disable, detect conflicts by room and time.
    - Training Groups: CRUD + disable, detect conflicts by group and time.
    - Instructors: CRUD + disable (permanent/contractor), detect conflicts by instructor and time.
    - Schedule Blocks: CRUD + cancel, validate end>start, prevent conflicts.
    - Observations: Add to schedule blocks (text, author, date, severity: info/warning/critical).

    ## Architecture Requirements
    - Backend layers: cmd/api, internal/{config,domain,application,http,repository,infrastructure}. Strict dependency rules.
    - Frontend features: src/{app,features,components,services,hooks,types}. API calls outside components.
    - Database: English singular collection names (room, trainingGroup, etc.), own init scripts, seed, indexes.

    ## Quality Requirements
    - Backend unit tests for domain rules and conflict detection.
    - Frontend tests for API client and key UI interactions.
    - Executable build/test commands with cwd, run, purpose, expected exit code.
    - QA reports citing commands/files; AppSec reports citing specific controls.
    - Release readiness: Docker Compose, healthchecks, .env.example, deployment/rollback, reproducible startup.

    ## Acceptance Criteria
    - docker compose up --build starts all services.
    - Backend healthcheck verifies dependency reachability.
    - Full CRUD + conflict prevention via API.
    - Tests/commands executable from clean checkout.
    - No monolothic files; no Spanish technical names.
    - Final state only completed with verifiable QA/AppSec/release evidence.

  functional_requirements:
    - "Room management: Create, list, update, disable rooms with code, name, capacity, location, active status."
    - "Training group management: Create, list, update, disable groups with code, program name, start/end dates, active status."
    - "Instructor management: Create, list, update, disable instructors with full name, email, contract type (permanent/contractor), specialization, active status."
    - "Schedule block management: Create, list, update, cancel blocks with date, start/end time, room, group, instructor, learning activity. Validate end > start. Prevent conflicts for room, group, and instructor."
    - "Observations: Add observations to schedule blocks with text, author, creation date, severity (info, warning, critical)."
    - "Conflict detection: Enforce no double-booking for room, training group, or instructor within overlapping time ranges."

  non_functional_requirements:
    - "Backend must use layered Go architecture (cmd, internal/{config,domain,application,http,repository,infrastructure})."
    - "HTTP handlers must not access MongoDB directly; use repository interfaces."
    - "Domain code must not import HTTP, DB, or framework packages."
    - "Server must include read/write/idle/shutdown timeouts."
    - "CORS must not be wildcard in production mode."
    - "500 responses must not expose raw internal errors."
    - "Source files < 300 lines."
    - "Frontend must use feature-oriented React architecture (app, features, components, services, hooks, types)."
    - "API calls live outside React components; API client has timeout and structured error handling."
    - "UI copy in Spanish, code names in English."
    - "MongoDB collections use English singular names (room, trainingGroup, instructor, scheduleBlock, observation)."
    - "Database init/seed scripts stored in database/ folder only, not in backend/."
    - "Include backend unit tests for domain rules and conflict detection."
    - "Include frontend tests for API client and key UI interactions."
    - "Test commands executable from clean checkout with documented cwd, run, purpose, expected exit code."
    - "QA reports must cite commands, exit codes, real files."
    - "AppSec reports must cite concrete files, lines/controls."
    - "Release readiness must verify Docker Compose, healthchecks, .env.example, deployment, rollback, reproducible startup."

  user_stories:
    - "As a schedule manager, I want to create and manage rooms so that I can assign physical spaces to schedule blocks."
    - "As a schedule manager, I want to create and manage training groups so that I can assign cohorts to schedule blocks."
    - "As a schedule manager, I want to create and manage instructors so that I can assign qualified staff to schedule blocks."
    - "As a schedule manager, I want to create schedule blocks with room/group/instructor assignments, and have the system prevent conflicts."
    - "As a schedule manager, I want to add observations to schedule blocks (info, warning, critical) to communicate issues."
    - "As a schedule manager, I want to see a healthcheck endpoint that confirms the backend is connected to the database."

  acceptance_criteria:
    - "docker compose up --build starts database, backend, and frontend containers."
    - "Backend healthcheck returns 200 only when MongoDB is reachable."
    - "Frontend can list and create rooms, instructors, training groups, schedule blocks, and observations through backend API."
    - "Conflict rules prevent double-booking a room, instructor, or training group for overlapping time ranges."
    - "All tests (backend unit tests, frontend tests) pass when executed from a clean checkout with provided commands."
    - "No component is delivered as a monolithic file (each project has proper subfolders)."
    - "No technical code or database schema object uses Spanish names (e.g., collection names are English singular)."
    - "Final state is not marked 'completed' unless QA, AppSec, and release gates pass with verifiable evidence (command outputs, file citations)."

  scope:
    in_scope:
      - "Room CRUD and conflict detection"
      - "Training group CRUD and conflict detection"
      - "Instructor CRUD (with permanent/contractor types) and conflict detection"
      - "Schedule block CRUD, validation (end > start), and conflict prevention"
      - "Observation creation on schedule blocks with severity levels"
      - "Layered Go backend with MongoDB"
      - "Feature-oriented React frontend with Spanish UI"
      - "Docker Compose orchestration with healthchecks"
      - "Unit tests for domain rules and conflict detection"
      - "Frontend tests for API client and key interactions"
      - "Seed data and initialization scripts in English"
    out_of_scope:
      - "User authentication and authorization (no login/roles)"
      - "Real-time notifications or email alerts"
      - "Reporting or analytics dashboards"
      - "Multi-language support (only Spanish UI)"
      - "Mobile or responsive design optimization"
      - "Integration with external calendars (e.g., Google Calendar)"
      - "Automatic scheduling or optimization algorithms"
      - "Audit logging beyond observation history"

  stakeholders:
    - "SENA academic coordinators and schedule managers"
    - "Instructors (end users who view and are assigned to schedules)"
    - "SENA IT operations team (deploy and maintain the system)"
    - "Quality assurance team (verify requirements and tests)"
    - "Security team (review AppSec controls)"
    - "Development team (implement and deliver the MVP)"

  quality_score: 92
```