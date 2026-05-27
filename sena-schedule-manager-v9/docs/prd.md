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

Any generated Node/Express backend, PostgreSQL/MySQL database, Angular/Vue frontend, or stack drift must be rejected.

## Component Boundaries

The workspace must be delivered as three independent projects plus root orchestration:

- `database/`: MongoDB init, seed, indexes, validation, Dockerfile or image configuration, and database-specific documentation.
- `backend/`: Go API only. It must not embed database init, seed, migrations, or Mongo scripts.
- `frontend/`: React UI only.
- root: `docker-compose.yml`, `.env.example`, `.gitignore`, and cross-component run documentation.

Each component must have its own container boundary. The database, backend, and frontend must not share a container.

## Functional Scope

### Rooms

- Create, list, update, and disable rooms.
- Store room code, name, capacity, location, and active status.
- Detect schedule conflicts by room and time range.

### Training Groups

- Create, list, update, and disable training groups.
- Store group code, program name, start date, end date, and active status.
- Detect schedule conflicts by group and time range.

### Instructors

- Create, list, update, and disable instructors.
- Store full name, email, contract type, specialization, and active status.
- Contract type must support permanent and contractor instructors.
- Detect schedule conflicts by instructor and time range.

### Schedule Blocks

- Create, list, update, and cancel schedule blocks.
- Store date, start time, end time, room, training group, instructor, and learning activity.
- Validate that end time is after start time.
- Prevent conflicts for room, training group, and instructor.

### Observations

- Add observations to schedule blocks.
- Store text, author, creation date, and severity.
- Severity must support info, warning, and critical.

## Architecture Requirements

### Backend

The Go backend must use layered architecture:

- `cmd/api` for bootstrap and wiring only.
- `internal/config` for configuration loading.
- `internal/domain` for entities, value objects, and domain rules.
- `internal/application` for use cases and service orchestration.
- `internal/http` for handlers, routes, request DTOs, and response DTOs.
- `internal/repository` for persistence interfaces or contracts.
- `internal/infrastructure` for MongoDB adapters and external details.

Rules:

- `main.go` must only compose dependencies and start the server.
- HTTP handlers must not access MongoDB directly.
- Application services/use cases must depend on repository interfaces, not concrete MongoDB adapters.
- Domain code must not import HTTP, MongoDB, Docker, environment, or framework packages.
- Repositories/adapters must not contain HTTP logic.
- Server must include read, write, idle, and shutdown timeouts.
- CORS must not be wildcard in production mode.
- 500 responses must not expose raw internal errors.
- Source files should stay under 300 lines.

### Frontend

The React frontend must use feature-oriented architecture:

- `src/app` for app composition and routing.
- `src/features` for feature modules.
- `src/components` for shared UI components.
- `src/services` for API clients.
- `src/hooks` for reusable hooks.
- `src/types` for shared contracts.

Rules:

- `App.jsx` or `App.tsx` must only compose routing/layout.
- API calls must live outside React components.
- API client must define a timeout and structured error handling.
- Local/demo persistence must not live in `App.jsx` or feature pages.
- Pages, components, and CSS files should stay under 300 lines.
- The UI copy must be Spanish, but implementation names must be English.

### Database

MongoDB collections and attributes must use English singular naming.

Allowed collection examples:

- `room`
- `trainingGroup`
- `instructor`
- `scheduleBlock`
- `observation`

Rejected collection examples:

- `rooms`
- `ambientes`
- `fichas`
- `instructores`
- `horarios`
- `observaciones`

Database project requirements:

- Own `database/` folder.
- Own init scripts, indexes, validation rules, seed data, and README.
- Seed data must use English field names.
- No database init or seed script may be stored under `backend/`.

## Quality Requirements

- Include backend unit tests for domain rules and application conflict detection.
- Include backend repository or adapter tests where practical.
- Include frontend tests for API client behavior and key UI interactions.
- Include executable build/test commands with `cwd`, `run`, `purpose`, and `expected_exit_code`.
- Persist command results in the runtime ledger.
- QA reports must cite commands, exit codes, and real files.
- AppSec reports must cite concrete files, lines or controls, not generic prose.
- Release readiness must verify Docker Compose, Dockerfiles, healthchecks, `.env.example`, deployment, rollback, and reproducible startup.

## Acceptance Criteria

- `docker compose up --build` starts database, backend, and frontend.
- Backend healthcheck returns healthy only when dependencies are reachable.
- Frontend can list and create rooms, instructors, training groups, schedule blocks, and observations through the backend API.
- Conflict rules prevent double-booking room, instructor, or training group.
- Tests/build commands are executable from clean checkout.
- No component is delivered as a monolithic file.
- No technical code or database schema object uses Spanish names.
- Final state must not be `completed` unless QA, AppSec, and release gates pass with verifiable evidence.
