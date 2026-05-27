# PRD - SENA Schedule Manager v8

## Product Context

SENA Schedule Manager is an internal scheduling product for training center operations. The product must help coordinators manage learning environments, schedules, training groups, instructors, instructor contract type, and operational observations. The goal of this v8 fire test is not only to create a functional MVP, but to verify that the Agentic SDLC OS enforces architecture, quality, security, and release readiness mechanically.

## Required Stack

- Backend: Go
- Frontend: React
- Database: MongoDB
- Runtime: Docker and Docker Compose

## Runtime Output Constraint

When an agent must emit structured data for the Agentic SDLC OS runtime, it must prefer strict JSON over YAML. Do not use Markdown code formatting inside structured scalar values. Every string value must be quoted when using JSON or YAML. This constraint exists only to keep runtime parsing deterministic during the v8 fire test.

## Non-Negotiable Engineering Rules

- MVP reduces functional scope, never architecture, security, tests, observability, or release standards.
- The workspace must be split into three independent projects: database/, backend/, and frontend/.
- Each component must own its container boundary and Dockerfile.
- Database initialization, indexes, validation, seed data, or migrations must live in database/, never inside backend/.
- Technical code, database identifiers, API contracts, file names, tests, comments, diagrams, and architecture contracts must be in English.
- User-facing labels and product documentation may be in Spanish.
- Database collection names and attributes must be English, singular, and consistent.
- Backend must respect dependency inversion and clean architecture boundaries.
- Frontend must avoid monolithic pages/components and must keep API clients outside components.
- QA, AppSec, and release evidence must cite real commands, files, exit codes, and artifacts.

## Functional Scope

The MVP must support:

- Manage learning environments.
- Manage schedules.
- Manage training groups.
- Manage instructors.
- Distinguish staff instructors and contractor instructors.
- Register operational observations linked to schedules or instructors.
- Expose basic CRUD workflows needed by coordinators.

## User Roles

- Academic coordinator: manages schedules, environments, groups, instructors, and observations.
- Instructor: views assigned schedules and related observations.
- Administrator: validates operational data and system readiness.

## Functional Requirements

1. The system must allow coordinators to create, update, list, and deactivate learning environments.
2. The system must allow coordinators to create, update, list, and deactivate training groups.
3. The system must allow coordinators to create, update, list, and deactivate instructors.
4. The system must classify instructors by contract type: staff or contractor.
5. The system must allow coordinators to create and update schedules linking environment, training group, instructor, day, time range, and notes.
6. The system must prevent obviously invalid schedules such as missing instructor, missing environment, or invalid time range.
7. The system must allow observations to be registered and associated with a schedule, instructor, or operational context.
8. The frontend must provide usable flows for the base entities without depending on mock-only behavior.
9. The backend must expose HTTP APIs with validation, error normalization, and repository abstractions.
10. The database project must define MongoDB collections, indexes, validation rules, and seed data independently from backend startup.

## Non-Functional Requirements

- Build must be reproducible from a clean checkout.
- Backend tests must run through a documented command.
- Frontend tests and production build must run through documented commands.
- Docker Compose must start database, backend, and frontend as separate services.
- Services must include healthchecks or equivalent readiness checks.
- Backend HTTP server must use timeouts.
- API clients must use timeouts.
- CORS must not be open by default.
- Error responses must not leak raw internal errors.
- AppSec evidence must reference concrete controls and files.
- Release readiness must include deployment and rollback commands.

## Acceptance Criteria

- The generated workspace contains database/, backend/, and frontend/.
- database/ contains MongoDB initialization artifacts, indexes, validation, and seed data using English singular names.
- backend/ contains Go code organized by domain, application/use cases, HTTP transport, repository interfaces/implementations, and configuration.
- frontend/ contains React code organized by app composition, features, components, services, hooks, and tests.
- No source file that implements application logic exceeds 300 lines.
- main.go is bootstrap and wiring only.
- App.jsx or equivalent app entrypoint is composition/routing only.
- Tests exist and commands are recorded with exit codes.
- QA, AppSec, and release gates cannot pass with narrative-only evidence.
- Final state can be completed only after release readiness passes.
