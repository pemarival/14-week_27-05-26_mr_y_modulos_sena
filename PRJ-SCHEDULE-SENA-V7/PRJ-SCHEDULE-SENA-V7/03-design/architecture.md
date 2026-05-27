# Architecture

## Components
1. **Frontend**: React SPA for user interaction.
2. **Backend**: Express API server serving JSON.
3. **Database**: PostgreSQL for persistent storage.

## Communication
Frontend communicates with Backend via RESTful HTTP calls over JSON.
Backend communicates with Database via SQL over TCP/IP using `pg` driver.

## Responsibilities
- Frontend: Rendering UI, state management, API integration.
- Backend: Routing, validation, business logic, persistence logic.
- Database: Data integrity, relational constraints.
