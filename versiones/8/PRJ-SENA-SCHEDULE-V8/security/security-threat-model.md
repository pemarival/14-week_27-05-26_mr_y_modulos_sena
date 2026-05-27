```yaml
security_threat_model:
  abuse_cases:
  - description: An unauthorized user attempts to create, update, or delete schedules,
      instructors, environments, or training groups.
    mitigation_strategy: "Implement API Gateway authentication and authorization checks. \
      Backend `transport` layer must enforce role-based access control (RBAC) on \
      all endpoints based on the `user_roles` specified in the PRD (Academic Coordinator, \
      Administrator, Instructor). For instance, only Coordinators/Admins can create/update/delete \
      schedules or instructors."
    name: Unauthorized Data Manipulation
  - description: An instructor attempts to view schedules or observations not assigned
      to them, or modify their own schedules.
    mitigation_strategy: "Implement granular, attribute-based access control (ABAC) \
      or resource-based authorization. The `application/usecase` layer must verify \
      that the authenticated `instructor_id` matches the `instructor_id` on the requested \
      schedule or observation before allowing read/update access."
    name: Privilege Escalation / Data Access Violation (Instructor)
  - description: Malicious user injects script or SQL/NoSQL commands into input fields
      (e.g., `name`, `description`, `email`, `expertise_area`) to compromise data
      or application.
    mitigation_strategy: "Implement robust input validation and sanitization at the \
      `infrastructure/transport` layer (for DTOs) and within `domain/entity` for \
      business rules. Use parameterized queries/ORM features of Go MongoDB driver. \
      Escape all output rendered in the frontend. Apply JSON Schema validation at \
      MongoDB level."
    name: Injection Attack
  - description: An attacker crafts requests to exploit vulnerabilities in environment
      or schedule creation, leading to resource exhaustion or denial of service (DoS).
    mitigation_strategy: "Implement rate limiting on API endpoints, especially creation/update \
      endpoints. Enforce strict data validation for numerical inputs (e.g., `capacity`, \
      `student_count`) and date ranges (e.g., `start_time`, `end_time`) to prevent \
      excessive resource allocation or infinite loops in scheduling logic. Implement \
      HTTP timeouts and database query timeouts."
    name: Resource Exhaustion / DoS
  - description: Access to PII fields (`instructor.name`, `instructor.email`, `observation.observed_by`)
      by unauthorized users or through insecure channels.
    mitigation_strategy: "Enforce strict access controls on PII fields via RBAC/ABAC. \
      Encrypt PII at rest in MongoDB. Mask PII in all logs. Use secure communication \
      (HTTPS). Ensure frontend does not cache PII in insecure storage (e.g., `localStorage`)."
    name: PII Exposure
  attack_surfaces:
  - description: Frontend application running in user browsers, interacting with the
      backend API.
    name: Frontend (React UI)
  - description: Backend API endpoints (`infrastructure/transport`) exposed to the
      frontend and potentially other internal services. This is the primary entry
      point for all business logic.
    name: Backend API Gateway / Transport Layer
  - description: MongoDB database, including collections and underlying file system.
      Accessible only by the backend.
    name: Database (MongoDB)
  - description: Docker daemon and container orchestration (Docker Compose).
    name: Container Runtime / Orchestration
  quality_score: 95
  required_controls:
  - control: Ensure all input data is validated and sanitized at the API boundary
      and within the domain layer.
    owner: Backend Team
    verification: "Implement DTO validation in `infrastructure/transport`. Domain \
      entities (`domain/entity`) must have validation methods. MongoDB JSON Schema \
      validation (`database/schema-validation`) must be applied. Unit and integration \
      tests for input validation scenarios."
  - control: Implement authentication for all API endpoints.
    owner: Backend Team
    verification: "All `infrastructure/transport` handlers must require an authentication \
      token. Automated API tests for unauthorized access (`401 Unauthorized`) on \
      all endpoints."
  - control: Implement role-based access control (RBAC) for all sensitive operations.
    owner: Backend Team
    verification: "Authorization logic in `application/usecase` and `infrastructure/transport` \
      must verify user roles (Academic Coordinator, Administrator, Instructor) against \
      action. Automated API tests for forbidden access (`403 Forbidden`) based on \
      role."
  - control: Encrypt PII data at rest in MongoDB.
    owner: Database Team / Backend Team
    verification: "MongoDB field-level encryption (FLE) for `instructor.name` and \
      `instructor.email`. Encryption keys must be managed securely (KMS). Auditable \
      proof of encryption at rest."
  - control: Mask PII in all application logs.
    owner: Backend Team
    verification: "Logging middleware must detect and mask `instructor.name` and \
      `instructor.email` before logging. Manual review of log samples and automated \
      tests for log masking."
  - control: Use HTTPS for all communication between frontend and backend.
    owner: Infrastructure Team
    verification: "Deployment configuration must enforce HTTPS. Certificate validity \
      check. Penetration testing to confirm no HTTP fallback."
  - control: Implement secure HTTP headers (e.g., HSTS, CSP, X-Frame-Options) in
      the backend.
    owner: Backend Team
    verification: "Response headers from `infrastructure/transport` must include \
      security headers. Automated security scanner (e.g., OWASP ZAP) reports."
  - control: Enforce strict CORS policies.
    owner: Backend Team
    verification: "CORS must be restricted to known frontend origins in `infrastructure/transport` \
      configuration. Manual testing of cross-origin requests from unauthorized domains."
  - control: Implement rate limiting on API endpoints to prevent abuse.
    owner: Backend Team
    verification: "Middleware in `infrastructure/transport` to limit requests per \
      IP/user. Performance tests to verify rate limiting behavior under load."
  - control: Ensure database connections use least privilege and strong authentication.
    owner: Database Team / Backend Team
    verification: "MongoDB user (`init-mongo.js`) must have only necessary permissions. \
      Strong passwords/mechanisms for DB access. Auditable proof of permissions. Connection \
      string must not be exposed."
  - control: Backend must run in a non-root container.
    owner: DevOps Team
    verification: "Dockerfile for backend must specify a non-root user. Runtime container \
      inspections (`docker inspect`)."
  - control: Frontend must avoid insecure client-side storage for sensitive data.
    owner: Frontend Team
    verification: "No `localStorage` or `sessionStorage` usage for PII or tokens. \
      Browser developer tools inspection. Automated vulnerability scans for client-side \
      storage."
  - control: Implement timeouts for all external calls (HTTP requests, DB queries).
    owner: Backend Team
    verification: "Go's `http.Client` and MongoDB driver client must have configured \
      timeouts. Code review and unit tests for timeout handling."
  - control: Ensure error messages exposed to clients are generic and do not leak
      sensitive information or stack traces.
    owner: Backend Team
    verification: "Error handling in `infrastructure/transport` must return generic \
      messages. Internal logs can contain full details. Penetration testing for error \
      message leakage."
  - control: Restrict the domain/core business logic from knowing persistence or
      transport details.
    owner: Backend Team
    verification: "Code review: `domain` and `application` packages must not import \
      `infrastructure/persistence` or `infrastructure/transport`. Static analysis \
      tools (e.g., `go vet`) and custom linters to enforce dependency inversion rules."
  - control: Implement logging and monitoring for security events and anomalies.
    owner: DevOps Team / Backend Team
    verification: "Centralized logging (e.g., ELK stack). Alerting for failed login \
      attempts, authorization failures, unusual traffic patterns. Regular review \
      of security logs."
  stride_threats:
  - description: An attacker could impersonate an Academic Coordinator to perform
      unauthorized actions on schedules, instructors, or environments.
    name: Spoofing Identity (S)
    mitigation_guidance: "Strong authentication and identity verification for users. \
      Implement JWTs for stateless authentication. Verify JWT signatures and expiration \
      on every request. Backend must validate `instructor_id` from token against \
      resource being accessed/modified to prevent impersonation."
  - description: An attacker could tamper with scheduled times, instructor assignments,
      or observation descriptions, leading to incorrect or malicious data.
    name: Tampering with Data (T)
    mitigation_guidance: "Input validation and sanitization. Robust authorization \
      checks (RBAC/ABAC) to ensure only authorized users can modify data. Implement \
      data integrity checks, e.g., enforce MongoDB JSON Schema validation. Log all \
      critical data modifications for auditability."
  - description: Unauthorized users could gain access to PII (instructor names, emails)
      or sensitive operational observations.
    name: Information Disclosure (I)
    mitigation_guidance: "Data encryption at rest (MongoDB FLE) and in transit (HTTPS). \
      Strict access control (RBAC/ABAC). Mask PII in logs. Avoid returning unnecessary \
      data in API responses. Frontend must not store PII in insecure client-side \
      storage."
  - description: An attacker could modify application code or configuration files,
      or inject malicious code through input, leading to unauthorized behavior or
      system compromise.
    name: Repudiation (R)
    mitigation_guidance: "Comprehensive logging of all critical actions with user \
      ID, timestamp, and action details. Secure build and deployment pipeline. Code \
      signing. Version control for all code and configurations. Regular security \
      audits."
  - description: An attacker could make the service unavailable by overwhelming it
      with requests or exploiting vulnerabilities that crash the application/database.
    name: Denial of Service (D)
    mitigation_guidance: "Rate limiting on API endpoints. Input validation for resource-intensive \
      operations. Implement timeouts for long-running processes or database queries. \
      Ensure container resource limits are set. Robust error handling to prevent \
      crashes. Health checks and automated restarts for services."
  - description: An attacker could elevate their privileges, e.g., an Instructor
      account gaining Academic Coordinator access, or bypassing normal authorization
      checks.
    name: Elevation of Privilege (E)
    mitigation_guidance: "Granular access control (RBAC/ABAC) strictly enforced \
      at the `application/usecase` layer. Principle of least privilege for all users \
      and services. Regular security audits and penetration testing to identify \
      privilege escalation paths."
  threat_model_document: "El sistema SENA Schedule Manager v8, construido con Clean \
    Architecture en Go (backend), React (frontend) y MongoDB (base de datos), requiere \
    un robusto modelo de seguridad. Las principales superficies de ataque incluyen \
    el frontend (interacciones del usuario), el backend API (puntos de entrada a la \
    lógica de negocio), la base de datos (persistencia de datos sensibles) y el \
    tiempo de ejecución de contenedores. Las amenazas STRIDE se evalúan para garantizar \
    la confidencialidad, integridad y disponibilidad. Los controles de seguridad \
    se centran en la autenticación, autorización granular (RBAC/ABAC), validación \
    de entradas, cifrado de PII en reposo y en tránsito, enmascaramiento de PII \
    en logs, CORS estricto, rate limiting, gestión de secretos, ejecución de contenedores \
    sin privilegios y logging/auditoría. Los casos de abuso clave incluyen manipulación \
    de datos no autorizada, escalada de privilegios, inyección y exposición de PII. \
    Se exige la implementación de timeouts HTTP/DB y mensajes de error genéricos. \
    La separación de preocupaciones inherente a Clean Architecture ayuda a segmentar \
    los controles de seguridad."
```