```yaml
security_threat_model:
  threat_model_document: >
    The SENA Schedule Manager follows Clean Architecture with a Go backend,
    React frontend, and MongoDB. The system manages PII (instructor email, phone,
    document_id; observation author) under Colombian Law 1581. The primary threat
    is unauthorized data access or modification via API injection, broken authentication,
    or misconfigured MongoDB. The frontend is restricted (no localStorage, sessionStorage,
    fetch in main.tsx) to reduce client-side data leaks. The backend layers enforce
    strict dependency inversion; domain entities are pure Go structs without transport
    or persistence metadata. All HTTP endpoints must validate and sanitize input.
    MongoDB replica set provides HA but requires TLS and auth. Conflict detection
    logic in schedule blocks is business-critical; injection in query filters could
    bypass checks. The Docker deployment must run containers as non-root and restrict
    network exposure.
  attack_surfaces:
    - name: HTTP API endpoints (Go handlers)
      description: Handlers for CRUD operations on rooms, groups, instructors, schedule blocks, observations. No authentication specified (high risk).
    - name: MongoDB direct access
      description: Database exposed internally; if compromised, full data access. Needs TLS and authentication.
    - name: Frontend React app
      description: Client-side code, bundled and served. XSS risks if user input rendered unsanitized. API calls in services layer.
    - name: Docker Compose network
      description: Inter-service communication; if one container breached, lateral movement possible.
    - name: Backup/archive storage
      description: Daily dumps and 5-year cold storage; weak access controls could leak PII.
  stride_threats:
    - threat: Spoofing
      category: Spoofing
      component: HTTP API
      description: "No authentication mechanism specified; an attacker could impersonate a valid user."
    - threat: Tampering
      category: Tampering
      component: API Input Validation
      description: "Unsantized input in conflict detection queries (e.g., room_id, start_time) could lead to NoSQL injection and data corruption."
    - threat: Repudiation
      category: Repudiation
      component: Audit Logging
      description: "Lack of audit logs for CRUD operations (especially deactivation/cancellation) allows repudiation of actions."
    - threat: Information Disclosure
      category: Information Disclosure
      component: Instructor/Observation data
      description: "PII fields (email, phone, document_id, author) stored in MongoDB without encryption at rest; exposed in API responses if not filtered."
    - threat: Denial of Service
      category: Denial of Service
      component: Conflict Detection Queries
      description: "Complex overlapping queries without rate limiting or query timeout could exhaust database connections."
    - threat: Elevation of Privilege
      category: Elevation of Privilege
      component: API Authorization
      description: "Assumes all API consumers have same permissions; no role separation (admin vs viewer) enables privilege escalation."
  required_controls:
    - control: Authenticate all API requests
      owner: backend/http (middleware)
      verification: "Integration test that returns 401 for unauthenticated requests to any protected endpoint."
    - control: Authorize based on role (admin/viewer)
      owner: backend/http (middleware)
      verification: "Test that a viewer cannot perform DELETE or UPDATE on rooms/instructors/schedule blocks."
    - control: Encrypt PII at rest (application-level encryption)
      owner: backend/infrastructure (MongoDB repository)
      verification: "Unit test that stored PII fields (email, phone, document_id, author) are encrypted before persisting and decrypted on read."
    - control: Input validation and sanitization
      owner: backend/http (DTOs) and backend/application (services)
      verification: "Test that injection payloads (e.g., $ne, $gt) in query parameters are rejected or sanitized."
    - control: Rate limiting on API endpoints
      owner: backend/http (middleware)
      verification: "Load test that after threshold, requests are throttled with 429 status."
    - control: Audit logging for create/update/delete and deactivation/cancellation
      owner: backend/infrastructure (logging) and application services
      verification: "Test that a log entry is written with timestamp, user, action, and entity ID for each mutating operation."
    - control: TLS for MongoDB connections
      owner: backend/infrastructure (MongoDB client config) and database (MongoDB server)
      verification: "Inspect Docker Compose or config for mongodb+srv URI and tls=true; integration test that connection fails without TLS."
    - control: Sanitize error messages (no stack traces in responses)
      owner: backend/http (middleware)
      verification: "Test that a 500 error returns generic error message, not internal details."
    - control: Restrict CORS to trusted origins
      owner: backend/http (middleware)
      verification: "Test that a request from non-whitelisted origin receives CORS error."
    - control: Set timeouts for HTTP handlers and database queries
      owner: backend/http (server config) and backend/infrastructure (MongoDB session)
      verification: "Unit test that context deadline exceeds after configured timeout."
    - control: Run containers as non-root user
      owner: DevOps (Dockerfiles)
      verification: "Dockerfile inspection: USER directive present; integration test that container process does not run as root."
    - control: Ensure domain/core layer has no dependency on transport/persistence metadata
      owner: backend/domain (Go structs)
      verification: "Static analysis: domain package imports zero MongoDB or HTTP packages."
  abuse_cases:
    - case: "Attacker enumerates schedule blocks by varying start_time in conflict API to infer future schedules."
    - case: "Former employee (author of observations) requests deletion of PII, but system lacks data deletion workflow."
    - case: "Attacker crafts a schedule block with end_time before start_time to trigger panic in validation logic."
    - case: "Unbounded observation text causes buffer overflow or stors large documents (denial of storage)."
    - case: "Bulk export of all instructors via repeated GET requests without pagination limits."
  quality_score: 0.85
```