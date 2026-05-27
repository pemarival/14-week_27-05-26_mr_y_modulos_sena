# Security Threat Model

| Vector | Description | Control | Severity |
|--------|-------------|---------|----------|
| SQL Injection | Attacker injects malicious SQL via inputs | Use parameterized queries via `pg` | High |
| XSS | Attacker injects scripts via observations | React escapes output by default. Validate input. | High |
| CSRF | Cross-site request forgery | Use proper CORS config and SameSite cookies | Medium |
| Unauthenticated Access | Accessing API without auth | (Future MVP) JWT authentication | High |

*Note: For this initial MVP, strict network bounds and CORS are relied upon for access control as auth is out of scope for the base DB test.*
