# Database Design

## Engine
PostgreSQL 16+

## Physical Types
- IDs: `UUID` (gen_random_uuid)
- Strings: `VARCHAR` or `TEXT`
- Times: `TIME` and `TIMESTAMP`
- Numbers: `INT`

## Indexes
- `idx_schedule_instructor`: On `schedule(instructor_id)`
- `idx_schedule_classroom_time_slot`: Unique constraint `(classroom_id, time_slot_id)`
- `idx_observation_schedule`: On `observation(schedule_id)`

## RTO/RPO
- RTO: 4 hours
- RPO: 24 hours (Daily backups)
