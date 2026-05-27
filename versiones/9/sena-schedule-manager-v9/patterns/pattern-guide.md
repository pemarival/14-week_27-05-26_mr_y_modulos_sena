```yaml
architecture:
  adr_document: |
    # ADR-001: Decision arquitectonica - Clean Architecture

    ## Contexto
    El proyecto SENA Schedule Manager requiere un backend Go con capas estrictas (cmd/api, internal/{config,domain,application,http,repository,infrastructure}) y un frontend React con estructura orientada a funcionalidades. El dominio es de complejidad moderada: CRUD de entidades (salas, instructores, grupos, bloques, observaciones) con reglas de conflicto por superposición de horarios. La vida útil del producto es a largo plazo (producción real), con alta exigencia de testeabilidad, mantenibilidad y sustitución de adaptadores.

    ## Decision
    Se adopta **Clean Architecture** (Robert C. Martin) como patrón primario para el backend. Las capas siguen la regla de dependencia hacia adentro:
    - Dominio: entidades y reglas de negocio puras (sin importaciones de frameworks, HTTP ni base de datos).
    - Aplicación: casos de uso (orquestan dominio).
    - Transporte (HTTP): handlers, DTOs y enrutamiento.
    - Persistencia: repositorios como interfaces en `internal/repository` e implementaciones concretas en `internal/infrastructure`.
    - Infraestructura: configuración, MongoDB cliente, logging, etc.

    ## Consecuencias
    - Alta testeabilidad: dominio y aplicación se prueban sin infraestructura.
    - Sustitución de adaptadores (base de datos, framework web) sin afectar lógica de negocio.
    - Curva de aprendizaje inicial para el equipo, compensada por claridad a largo plazo.
    - Riesgo de overengineering: se mitiga manteniendo el alcance MVP y no introduciendo abstracciones prematuras (ej: repositorio genérico).

    ## Estructura de capas (nivel 3)
    ```
    backend/
      cmd/api/
        main.go
      internal/
        config/
          config.go, config_test.go
        domain/
          room.go, room_test.go
          instructor.go, instructor_test.go
          traininggroup.go, traininggroup_test.go
          scheduleblock.go, scheduleblock_test.go
          observation.go, observation_test.go
        application/
          room_service.go, room_service_test.go
          instructor_service.go, instructor_service_test.go
          traininggroup_service.go, traininggroup_service_test.go
          scheduleblock_service.go, scheduleblock_service_test.go
          observation_service.go, observation_service_test.go
        http/
          handler.go
          room_handler.go, room_handler_test.go
          instructor_handler.go, instructor_handler_test.go
          traininggroup_handler.go, traininggroup_handler_test.go
          scheduleblock_handler.go, scheduleblock_handler_test.go
          observation_handler.go, observation_handler_test.go
          dto.go
          middleware.go
        repository/
          room_repository.go
          instructor_repository.go
          traininggroup_repository.go
          scheduleblock_repository.go
          observation_repository.go
        infrastructure/
          mongodb/
            client.go
            room_repository_mongo.go
            instructor_repository_mongo.go
            traininggroup_repository_mongo.go
            scheduleblock_repository_mongo.go
            observation_repository_mongo.go
    ```

    ## Reglas de dependencia (estrictas)
    - `domain` -> no importa nada (solo stdlib Go y errores)
    - `application` -> depende de `domain` e interfaces en `repository`
    - `http` -> depende de `application` y `domain` (DTOs)
    - `repository` -> solo interfaces; depende de `domain`
    - `infrastructure` -> depende de `repository` y `domain`
    - `cmd/api` -> depende de `config`, `http`, `infrastructure` (wiring)
    - Ninguna capa puede importar hacia afuera (ej: `http` no puede importar `infrastructure`). Violaciones detectables con `grep`.

  patterns:
    - Clean Architecture

  quality_score: 0.90

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
        - init
        - seed
        - Dockerfile
      source_extensions:
        - .sh
        - .js
      test_patterns:
        - "*.test.sh"
      minimum_source_files: 1

    backend:
      root: backend
      required_paths:
        - Dockerfile
        - cmd/api
        - internal/domain
        - internal/application
        - internal/http
        - internal/repository
        - internal/infrastructure
      source_extensions:
        - .go
      test_patterns:
        - "*_test.go"
      minimum_source_files: 4
      entrypoints:
        - path: backend/cmd/api/main.go
          purpose: Bootstrap y wiring solamente
          forbidden_patterns:
            - mongodb\.Connect
            - .*repository\.New.*Mongo.*
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
        - src/types
      source_extensions:
        - .ts
        - .tsx
      test_patterns:
        - "*.test.ts"
        - "*.test.tsx"
      minimum_source_files: 4
      entrypoints:
        - path: frontend/src/app/main.tsx
          purpose: Composicion/routing solamente
          forbidden_patterns:
            - localStorage
            - sessionStorage
            - fetch(
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

  # --- CHECKLIST DE VALIDACION CLEAN ARCHITECTURE ---
  # 1. Verificar que domain/ no importe paquetes de infraestructura:
  #    grep -r "import.*infrastructure\|import.*mongo\|import.*http" backend/internal/domain/
  #    HALT si encuentra.
  # 2. Verificar que http/ no importe repository o infrastructure:
  #    grep -r "import.*repository\|import.*infrastructure" backend/internal/http/
  #    HALT si encuentra.
  # 3. Verificar que application/ solo importe domain y repository interfaces:
  #    grep -r "import.*infrastructure\|import.*http\|import.*mongo" backend/internal/application/
  #    HALT si encuentra.
  # 4. Verificar que cmd/api/main.go no contenga lógica de negocio (máx 3 funciones):
  #    cat backend/cmd/api/main.go | grep -c "^func" | read count; [ "$count" -gt 3 ] && HALT
  # 5. Verificar que los nombres de colecciones MongoDB sean inglés singular:
  #    grep -r "Collection\(.*\)" database/ --include="*.js" | grep -v "room\|trainingGroup\|instructor\|scheduleBlock\|observation"
  #    HALT si hay nombres no permitidos.

  # --- SOLID APLICADO A CLEAN ARCHITECTURE ---
  # **Single Responsibility**: Cada capa tiene una única responsabilidad.
  #   - Cumplimiento: domain/ contiene solo entidades y reglas, no persistencia.
  #   - Violación a evitar: Poner validación de base de datos en las entidades del dominio.
  # **Open/Closed**: Abierto a extensión, cerrado a modificación.
  #   - Cumplimiento: Agregar un nuevo repositorio (ej: Cache) sin modificar casos de uso (solo implementar interfaz).
  #   - Violación: Si un caso de uso instancia directamente un repositorio concreto (ej: NewRoomRepositoryMongo) en lugar de recibir la interfaz.
  # **Liskov Substitution**: Las implementaciones de repositorios deben ser sustituibles.
  #   - Cumplimiento: RoomRepositoryMongo implementa RoomRepository interface; cualquier test puede usar un mock.
  #   - Violación: Lanzar errores específicos de MongoDB en la interfaz de repositorio (tipo no documentado).
  # **Interface Segregation**: Interfaces pequeñas por entidad.
  #   - Cumplimiento: Cada repositorio tiene métodos propios (ej: RoomRepository tiene solo Save, FindByID, FindAll, Delete).
  #   - Violación: Un repositorio genérico con métodos que no aplican a todas las entidades (ej: Cancel no aplica a Room).
  # **Dependency Inversion**: Las capas de alto nivel no dependen de implementaciones concretas.
  #   - Cumplimiento: application/ depende de repository.Interfaces, no de infrastructure/mongo.
  #   - Violación: http/ crea una instancia de repositorio concreto directamente (new RoomRepositoryMongo).
```