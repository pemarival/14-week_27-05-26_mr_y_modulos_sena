# Rollback Plan

## Triggers
- Critical P0 issue in production post-deployment.
- Database migration failure.

## Procedure
1. `docker compose down`
2. Revert to previous image tags in `docker-compose.yml`.
3. Restore database from pre-deployment backup.
4. `docker compose up -d`

## Estimated Time
15 minutes.

## Responsible
A11 Release Manager.
