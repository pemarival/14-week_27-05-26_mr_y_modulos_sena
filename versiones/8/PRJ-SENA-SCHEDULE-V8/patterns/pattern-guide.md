```yaml
architecture:
  adr_document: |
    # ADR-001: Decision Arquitectónica - Clean Architecture para SENA Schedule Manager v8

    ## Contexto

    El producto SENA Schedule Manager v8 es una aplicación interna para gestionar operaciones de un centro de formación. Incluye la administración de entornos de aprendizaje, horarios, grupos de formación, instructores y observaciones operacionales. El MVP busca funcionalidades básicas de CRUD, pero con un énfasis no negociable en la arquitectura, calidad, seguridad y preparación para el lanzamiento. El equipo es de tamaño moderado y se espera que el producto tenga una vida útil considerable con posibles extensiones futuras. Se requiere una división clara en `database/`, `backend/` y `frontend/`, con independencia de contenedores.

    ## Decisión

    Se ha seleccionado **Clean Architecture** como patrón arquitectónico primario para el backend (Go) y sus principios para el frontend (React).

    ### Justificación
    1.  **Dominio Complejo a Medio Plazo**: Aunque el MVP actual es un CRUD básico, el dominio de gestión de horarios y personal en un centro de formación puede crecer en complejidad (ej., optimización de horarios, gestión de recursos avanzados, integración con otros sistemas). Clean Architecture proporciona la flexibilidad necesaria para acomodar esta evolución sin reescrituras mayores.
    2.  **Mantenibilidad y Testeabilidad**: La estricta separación de capas y la inversión de dependencias garantizan que la lógica de negocio central (domain y application) sea completamente independiente de detalles externos como la base de datos o el framework web. Esto facilita el desarrollo de pruebas unitarias y de integración robustas, y simplifica el mantenimiento a largo plazo.
    3.  **Independencia de Frameworks y Base de Datos**: Uno de los principios clave de Clean Architecture es la independencia de los detalles externos. Esto permite, por ejemplo, cambiar la base de datos (aunque MongoDB es un requisito actual, la abstracción facilita un futuro cambio) o el framework web con un impacto mínimo en la lógica de negocio principal. Esto es crucial dado el requisito de "capas sustituibles" de Clean Architecture.
    4.  **Cumplimiento de NFRs y Reglas No Negociables**:
        *   "Backend must respect dependency inversion and clean architecture boundaries": Cumplimiento directo.
        *   "Frontend must avoid monolithic pages/components and must keep API clients outside components": Los principios de Clean Architecture se adaptan bien a la separación de preocupaciones en el frontend, manteniendo la lógica de negocio (hooks, servicios de dominio) separada de la UI.
        *   Asegura la calidad y la robustez del sistema desde el inicio, alineándose con el requisito de no reducir la arquitectura ni la calidad por el MVP.

    ### Descartes
    *   **DDD (Domain-Driven Design)**: Aunque apropiado para dominios muy complejos, para un MVP con CRUDs básicos podría introducir una sobrecarga inicial significativa. Los principios de DDD (Aggregates, Bounded Contexts) se pueden ir introduciendo gradualmente dentro de una Clean Architecture si el dominio crece en complejidad.
    *   **Hexagonal Architecture**: Muy similar a Clean Architecture, de hecho, Clean Architecture puede verse como una evolución de la Hexagonal. Clean Architecture ofrece una guía más detallada sobre las capas internas (Enterprise Business Rules, Application Business Rules). La elección de Clean es por su explicitud en el número y propósito de las capas.
    *   **Modular Monolith**: No aplica dado el requisito explícito de tener tres proyectos independientes (`database/`, `backend/`, `frontend/`). Además, no aborda directamente la estructura interna de cada componente de la misma manera que Clean Architecture.
    *   **CQRS (Command Query Responsibility Segregation)**: Demasiado complejo para un MVP con necesidades asimétricas de lectura/escritura aún no demostradas. Puede ser una evolución futura si los requisitos de rendimiento o escalabilidad de lecturas y escrituras divergen significativamente.

    ## Consecuencias

    ### Positivas
    *   **Alta Cohesión, Bajo Acoplamiento**: Cada capa tiene una responsabilidad clara, lo que facilita el entendimiento y la modificación.
    *   **Mayor Testeabilidad**: La lógica de negocio es fácilmente testeable de forma aislada, sin necesidad de infraestructura externa.
    *   **Flexibilidad Tecnológica**: Permite cambiar componentes de infraestructura (ej. ORM, framework web) con menor impacto.
    *   **Escalabilidad del Equipo**: Facilita que diferentes equipos trabajen en distintas capas o dominios con menos conflictos.

    ### Negativas
    *   **Curva de Aprendizaje Inicial**: Requiere un entendimiento sólido de los principios de Clean Architecture, especialmente la inversión de dependencias.
    *   **Boilerplate Code**: Puede introducir más archivos y abstracciones para tareas simples, lo que puede sentirse excesivo en un MVP.
    *   **Configuración Inicial Más Compleja**: El "wiring" o inyección de dependencias puede ser más elaborado al principio.

    ## Estructura de Capas y Restricciones

    ### Backend (Go) - Clean Architecture
    La dirección de las flechas indica la dirección permitida de las dependencias.

    ```
    infrastructure/
      ├─ transport/ (HTTP handlers, DTOs, routing)
      ├─ persistence/ (Repository implementations, DB clients, ORM)
      └─ services/ (External service clients, e.g., email sender)
          ↓
    application/
      ├─ usecase/ (Application-specific business rules, orchestrates domain entities)
      ├─ port/ (Interfaces for secondary actors - repositories, external services)
      └─ event/ (Application-specific events)
          ↓
    domain/
      ├─ entity/ (Enterprise-wide business rules, core entities)
      ├─ valueobject/ (Small objects that represent a conceptual whole)
      ├─ service/ (Domain services that encapsulate business logic involving multiple entities)
      └─ event/ (Domain events)
    ```

    **Reglas de Dependencia (Backend):**
    *   Las dependencias solo pueden apuntar hacia adentro (capas más internas). Por ejemplo, `application` puede depender de `domain`, pero `domain` NO puede depender de `application`.
    *   `infrastructure` depende de `application`.
    *   `application` depende de `domain`.
    *   `main.go` (en la raíz del backend) es el punto de entrada y se encarga del "wiring" (inyección de dependencias) y configuración inicial. No debe contener lógica de negocio.

    ### Frontend (React) - Principios de Clean Architecture
    Aunque React no impone una arquitectura estricta, aplicaremos los principios de Clean Architecture para organizar el código.

    ```
    frontend/
      ├─ src/
      │  ├─ app/ (Root application setup, routing, layout)
      │  ├─ features/ (Self-contained, domain-specific modules with components, hooks, services)
      │  │  ├─ [feature-name]/
      │  │  │  ├─ components/ (UI components specific to this feature)
    │  │  │  ├─ hooks/ (Custom hooks for feature logic)
    │  │  │  ├─ services/ (API client calls specific to this feature, interacts with backend ports)
    │  │  │  ├─ pages/ (Feature-specific pages, orchestrates components and hooks)
      │  ├─ shared/
      │  │  ├─ components/ (Reusable UI components across features)
      │  │  ├─ hooks/ (Reusable general-purpose hooks)
      │  │  ├─ services/ (General-purpose API client, authentication, utilities)
      │  │  ├─ utils/ (Utility functions)
      │  ├─ lib/ (External libraries or configurations)
      │  ├─ public/ (Static assets)
      │  ├─ index.js (Entry point)
    ```

    **Reglas de Dependencia (Frontend):**
    *   `features/` puede usar componentes, hooks, y servicios de `shared/`.
    *   `app/` orquesta `features/` y `shared/components`.
    *   Las páginas dentro de `features/[feature-name]/pages` orquestan los componentes y hooks de su propia característica, y pueden usar servicios de `shared/services` o `features/[feature-name]/services`.
    *   Los servicios (en `features/` o `shared/`) son la única capa que interactúa directamente con la API del backend. Los componentes y hooks NO deben realizar llamadas `fetch` directamente.

    ### Database (MongoDB)
    ```
    database/
      ├─ Dockerfile (MongoDB container setup)
      ├─ docker-compose.yml (Service definition, volumes)
      ├─ init-mongo.js (Initialization script, user creation)
      ├─ schema-validation/ (JSON Schema for collections)
      │  ├─ instructor.json
      │  ├─ environment.json
      │  ├─ group.json
      │  ├─ schedule.json
      │  ├─ observation.json
      ├─ migrations/ (Scripts for schema changes)
      ├─ seed/ (Initial data for collections)
      │  ├─ seed-instructors.js
      │  ├─ seed-environments.js
    ```

    **Reglas de Dependencia (Database):**
    *   El directorio `database/` es auto-contenido y no debe tener dependencias del `backend/` o `frontend/` para su operación de inicialización.
    *   Las validaciones de esquema (`schema-validation/`) son la única "lógica de negocio" que reside en la base de datos, asegurando la integridad de los datos a nivel de persistencia.

    ## Ejemplos para Entidades Clave (Backend - Go)

    ### 1. Instructor
    *   **domain/entity/instructor.go**: `Instructor` struct (ID, Name, Type, etc.), métodos para validación de reglas de negocio intrínsecas (ej. `func (i *Instructor) IsStaff() bool`).
    *   **application/port/instructor_repository.go**: Interface `InstructorRepository` (ej. `Save`, `FindByID`, `FindAll`).
    *   **application/usecase/instructor_usecase.go**: `InstructorUseCase` struct, `CreateInstructor(input CreateInstructorInput) (*Instructor, error)`, `GetInstructor(id string) (*Instructor, error)`. Orquesta `InstructorRepository`.
    *   **infrastructure/persistence/mongo/instructor_repository.go**: Implementación de `InstructorRepository` usando el cliente MongoDB.
    *   **infrastructure/transport/http/instructor_handler.go**: `CreateInstructorHandler(w http.ResponseWriter, r *http.Request)`, convierte DTOs a entidades de dominio y llama a `InstructorUseCase`.

    ### 2. Schedule
    *   **domain/entity/schedule.go**: `Schedule` struct (ID, StartTime, EndTime, InstructorID, GroupID, EnvironmentID, etc.), métodos de validación de solapamiento de horarios (`func (s *Schedule) OverlapsWith(other Schedule) bool`).
    *   **application/port/schedule_repository.go**: Interface `ScheduleRepository` (`Save`, `FindByID`, `FindByInstructorID`, etc.).
    *   **application/usecase/schedule_usecase.go**: `ScheduleUseCase` struct, `CreateSchedule(input CreateScheduleInput) (*Schedule, error)`, `AssignInstructorToSchedule(scheduleID, instructorID string) error`. Utiliza `ScheduleRepository`, `InstructorRepository`, `GroupRepository`, `EnvironmentRepository`.
    *   **infrastructure/persistence/mongo/schedule_repository.go**: Implementación de `ScheduleRepository`.
    *   **infrastructure/transport/http/schedule_handler.go**: `CreateScheduleHandler`, `GetScheduleHandler`.

    ### 3. Environment
    *   **domain/entity/environment.go**: `Environment` struct (ID, Name, Capacity, Type), métodos de negocio (ej. `func (e *Environment) IsAvailable(timeRange TimeRange) bool`).
    *   **application/port/environment_repository.go**: Interface `EnvironmentRepository` (`Save`, `FindByID`, `FindAll`).
    *   **application/usecase/environment_usecase.go**: `EnvironmentUseCase` struct, `CreateEnvironment(input CreateEnvironmentInput) (*Environment, error)`, `GetEnvironment(id string) (*Environment, error)`. Orquesta `EnvironmentRepository`.
    *   **infrastructure/persistence/mongo/environment_repository.go**: Implementación de `EnvironmentRepository`.
    *   **infrastructure/transport/http/environment_handler.go**: `CreateEnvironmentHandler`, `GetEnvironmentHandler`.

    ## Checklist de Validación Automática

    ### Backend (Go)
    *   **Estructura de Carpetas**:
        *   `grep -r "package domain" backend/application` -> NO DEBE ENCONTRAR NADA (Domain no puede depender de Application)
        *   `grep -r "package infrastructure" backend/application` -> NO DEBE ENCONTRAR NADA (Infrastructure no puede depender de Application)
        *   `grep -r "package infrastructure" backend/domain` -> NO DEBE ENCONTRAR NADA (Infrastructure no puede depender de Domain)
        *   `grep -r "package application" backend/domain` -> NO DEBE ENCONTRAR NADA (Application no puede depender de Domain)
    *   **Archivo `main.go`**:
        *   `grep -E "func main\(" backend/main.go | wc -l` -> DEBE SER 1
        *   `grep -E "logic\.DoSomething\(\)" backend/main.go` -> NO DEBE ENCONTRAR NADA (No lógica de negocio en main)
    *   **Interfaces en Application Ports**:
        *   `find backend/application/port -name "*_repository.go" | wc -l` -> DEBE SER > 0

    ### Frontend (React)
    *   **Separación de API Clients**:
        *   `grep -r "fetch(" frontend/src/features` -> NO DEBE ENCONTRAR NADA (Fetch solo en services o shared)
        *   `grep -r "axios." frontend/src/features` -> NO DEBE ENCONTRAR NADA (Axios solo en services o shared)
        *   `find frontend/src/shared/services -name "*.js*" | wc -l` -> DEBE SER > 0 (Asegura existencia de servicios compartidos)
    *   **Entrypoint `App.jsx`**:
        *   `grep -E "logic\.DoSomething\(\)" frontend/src/app/App.jsx` -> NO DEBE ENCONTRAR NADA (Solo composición/routing)
        *   `grep -E "useState" frontend/src/app/App.jsx | wc -l` -> DEBE SER < 5 (Pocos estados, principalmente para routing o layout global)

    ### HALT Criterios (Violaciones Graves)
    *   Si se encuentra una dependencia de una capa interna a una externa (ej., `domain` importando `application`).
    *   Si `main.go` o `App.jsx` contienen lógica de negocio significativa (más allá de bootstrapping/routing).
    *   Si no hay interfaces de repositorios en la capa `application/port` del backend.
    *   Si se detectan llamadas directas a `fetch` o `axios` fuera de la capa `services` en el frontend.

    ## SOLID Aplicado a Clean Architecture

    ### Single Responsibility Principle (SRP)
    *   **Cumplimiento**: Un `InstructorUseCase` (en `application/usecase`) es responsable únicamente de coordinar las operaciones relacionadas con los instructores (crear, obtener, actualizar, eliminar). No se encarga de la persistencia de datos (eso es responsabilidad de `InstructorRepository`) ni de la representación HTTP (eso es responsabilidad de `InstructorHandler`).
    *   **Violación a evitar**: Un `InstructorHandler` que directamente contiene lógica para guardar un instructor en la base de datos (mezclando responsabilidades de transporte y persistencia).

    ### Open/Closed Principle (OCP)
    *   **Cumplimiento**: Si necesitamos cambiar la forma en que se almacenan los instructores (ej. de MongoDB a PostgreSQL), solo necesitamos crear una nueva implementación de `InstructorRepository` (en `infrastructure/persistence/sql/instructor_repository.go`) sin modificar el `InstructorUseCase` ni el código del dominio. El sistema está abierto a extensión (nuevas implementaciones de repositorio) pero cerrado a modificación (código de negocio central).
    *   **Violación a evitar**: Un `InstructorUseCase` que tiene sentencias `if/else` o `switch` para manejar diferentes tipos de bases de datos, obligando a modificarlo cada vez que se añade una nueva persistencia.

    ### Liskov Substitution Principle (LSP)
    *   **Cumplimiento**: Si tenemos una interfaz `Notifier` (`application/port/notifier.go`) con un método `Send(message string) error`, podemos tener implementaciones `EmailNotifier` y `SMSNotifier`. El `ScheduleUseCase` puede usar cualquier implementación de `Notifier` sin saber cuál es, y el comportamiento del sistema seguirá siendo correcto.
    *   **Violación a evitar**: Si `EmailNotifier` al implementar `Send` requiere un `subject` y `SMSNotifier` no lo usa, o lo ignora. Esto violaría LSP porque `SMSNotifier` no podría sustituir a `EmailNotifier` sin un cambio en el comportamiento esperado por el cliente.

    ### Interface Segregation Principle (ISP)
    *   **Cumplimiento**: En lugar de tener una única interfaz `MegaRepository` con todos los métodos de CRUD para todas las entidades, tenemos interfaces específicas como `InstructorRepository` y `ScheduleRepository`, cada una con solo los métodos que le corresponden. Esto también se aplica a interfaces más finas, como `Creator` o `Finder` si la entidad tiene muchas operaciones.
    *   **Violación a evitar**: Una única interfaz `CrudOperations` con `Create(entity interface{})`, `Read(id string, entity interface{})`, `Update(entity interface{})`, `Delete(id string)`. Esto obligaría a cada repositorio a implementar métodos que quizás no necesite o no apliquen, o a lanzar errores como "NotImplemented".

    ### Dependency Inversion Principle (DIP)
    *   **Cumplimiento**: La capa `application` (que contiene los `usecase`s) depende de abstracciones (interfaces de repositorios en `application/port`), no de implementaciones concretas de la capa `infrastructure` (como `mongo/instructor_repository.go`). Las implementaciones concretas son inyectadas en los `usecase`s durante el "wiring" en `main.go`.
    *   **Violación a evitar**: Un `InstructorUseCase` que directamente importa y crea una instancia de `infrastructure/persistence/mongo/instructor_repository.go`. Esto acoplaría la lógica de negocio a un detalle de infraestructura, impidiendo la independencia de la base de datos y dificultando las pruebas.
  patterns:
    - Clean Architecture
  quality_score: 0.9
  separation_of_concerns:
    backend_root: backend
    domain: backend/domain
    application: backend/application
    transport: backend/infrastructure/transport
    persistence: backend/infrastructure/persistence
    database_root: database
    database_migrations: database/migrations
    database_seed: database/seed
    frontend_root: frontend
    frontend_app: frontend/src/app
    frontend_features: frontend/src/features
    frontend_services: frontend/src/shared/services
  architecture_contract:
    database:
      root: database
      required_paths:
        - migrations
        - seed
        - Dockerfile
        - init-mongo.js
      source_extensions:
        - .js
        - .json
      test_patterns:
        - "database/tests/**/*.js"
      minimum_source_files: 4
    backend:
      root: backend
      required_paths:
        - Dockerfile
        - domain
        - application
        - infrastructure/transport
        - infrastructure/persistence
      source_extensions:
        - .go
      test_patterns:
        - "backend/**/*_test.go"
      minimum_source_files: 10
      entrypoints:
        - path: main.go
          purpose: Bootstrap y wiring solamente
          forbidden_patterns:
            - "func (s *Service) "
            - "db.Collection("
            - "http.HandleFunc"
          max_function_declarations: 3
    frontend:
      root: frontend
      required_paths:
        - Dockerfile
        - src/app
        - src/features
        - src/shared/components
        - src/shared/services
      source_extensions:
        - .js
        - .jsx
        - .ts
        - .tsx
      test_patterns:
        - "frontend/**/*.test.js"
        - "frontend/**/*.test.jsx"
      minimum_source_files: 8
      entrypoints:
        - path: src/app/App.jsx
          purpose: Composicion/routing solamente
          forbidden_patterns:
            - localStorage
            - sessionStorage
            - "fetch("
            - "axios."
            - "db.Collection("
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
```