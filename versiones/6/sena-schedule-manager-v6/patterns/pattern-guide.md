```yaml
architecture:
  adr_document: |
    # ADR-001: Layered Go React Mongo Architecture

    Context: SENA Schedule Manager V6 is a small operational MVP, but it has
    enough domain rules to require clean boundaries: schedule time validation,
    instructor type, classroom capacity and observations linked to schedules.
    The runtime also requires database, backend and frontend to be validated as
    independent components.

    Decision: use Clean Architecture with Hexagonal boundaries for the backend,
    a feature-oriented React frontend, and a separate MongoDB database project.
    Backend dependencies point inward: transport adapters call application use
    cases, application coordinates domain and repository ports, domain remains
    pure, and repository infrastructure maps persistence documents to domain
    objects. Database initialization, indexes and seed data live only in the
    database component. Frontend composition and routing live in app, feature UI
    lives in features, reusable widgets live in components, and all API calls
    live in services with timeout handling.

    Consequences: this is more structure than a quick demo, but it prevents the
    failures observed in earlier fire tests: monolithic handlers/pages, Mongo
    metadata in domain entities, database seed embedded in backend, narrative QA
    evidence and release without rollback. MVP scope is reduced to core CRUD and
    validation flows, while architecture, tests, security and release readiness
    remain mandatory.

    Validation checklist: backend entrypoint is bootstrap only; domain/core has
    no persistence or transport metadata; handlers contain no direct database
    access; repositories contain no HTTP concerns; App.jsx is composition only;
    database owns init, seed and index artifacts; every component has a
    container boundary; QA, AppSec and release artifacts are produced before
    completion.
  patterns:
    - Clean Architecture
    - Hexagonal Architecture
    - SOLID
  quality_score: 0.91
  separation_of_concerns:
    backend_root: backend
    domain: backend/internal/domain
    application: backend/internal/application
    transport: backend/internal/http
    persistence: backend/internal/repository
    database_root: database
    database_migrations: database/init
    database_seed: database/seed
    frontend_root: frontend
    frontend_app: frontend/src/app
    frontend_features: frontend/src/features
    frontend_services: frontend/src/services
  architecture_contract:
    database:
      root: database
      required_paths:
        - Dockerfile
        - init
        - seed
        - tests
      source_extensions:
        - .js
        - .json
      test_patterns:
        - "*.test.js"
        - "validate-*.js"
      minimum_source_files: 2
    backend:
      root: backend
      required_paths:
        - Dockerfile
        - cmd/api
        - internal/config
        - internal/domain
        - internal/application
        - internal/http
        - internal/repository
      source_extensions:
        - .go
      test_patterns:
        - "*_test.go"
      minimum_source_files: 8
      entrypoints:
        - path: cmd/api/main.go
          purpose: Bootstrap and dependency wiring only
          forbidden_patterns:
            - "mongo.Connect"
            - "InsertOne"
            - "Find("
            - "CreateOne"
            - "seed"
            - "createIndexes"
          max_function_declarations: 3
    frontend:
      root: frontend
      required_paths:
        - Dockerfile
        - src/app
        - src/features
        - src/components
        - src/services
        - src/hooks
      source_extensions:
        - .js
        - .jsx
      test_patterns:
        - "*.test.jsx"
        - "*.test.js"
      minimum_source_files: 8
      entrypoints:
        - path: src/app/App.jsx
          purpose: Composition and routing only
          forbidden_patterns:
            - localStorage
            - sessionStorage
            - "fetch("
            - "axios."
          max_function_declarations: 5
  skill_selection:
    backend:
      - backend/senior-backend.md
      - backend/go.md
    frontend:
      - frontend/senior-frontend.md
      - frontend/react.md
    database:
      - database/mongo.md
    qa:
      - qa/contract-verification.md
    security:
      - security/api.md
  tech_stack:
    backend: Go
    frontend: React
    database: MongoDB
    runtime: Docker Compose
```
