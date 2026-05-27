```yaml
db_design:
  db_design_document: |
    # MongoDB Physical Design

    MongoDB is selected because the MVP needs simple document persistence for
    operational schedules and related catalogs, with low relational complexity
    and fast local setup. Collections remain normalized by aggregate root:
    classroom, student_group, instructor, schedule and observation. Schedule
    documents reference catalog identifiers instead of embedding catalogs so
    backend application rules can validate dependencies explicitly. The design
    preserves pure backend domain entities: BSON names, ObjectId conversion and
    document validation live in database init files and repository documents,
    never in domain/core. The database project owns init, seed, indexes and
    validation smoke scripts.
  engine: MongoDB
  physical_objects:
    - name: classroom
      type: collection
      purpose: "Store physical learning environments"
      fields:
        - _id
        - code
        - name
        - location
        - capacity
    - name: student_group
      type: collection
      purpose: "Store training groups"
      fields:
        - _id
        - code
        - program_name
        - learner_count
    - name: instructor
      type: collection
      purpose: "Store instructor catalog records"
      fields:
        - _id
        - document_number
        - full_name
        - email
        - instructor_type
    - name: schedule
      type: collection
      purpose: "Store planned training sessions"
      fields:
        - _id
        - classroom_id
        - student_group_id
        - instructor_id
        - day
        - start_time
        - end_time
        - status
    - name: observation
      type: collection
      purpose: "Store follow-up notes linked to schedule"
      fields:
        - _id
        - schedule_id
        - note
        - created_at
  indexes:
    - object: classroom
      fields:
        - code
      reason: "unique lookup and list ordering"
    - object: student_group
      fields:
        - code
      reason: "unique lookup and list ordering"
    - object: instructor
      fields:
        - document_number
      reason: "unique instructor identity lookup"
    - object: schedule
      fields:
        - day
        - classroom_id
      reason: "detect classroom schedule conflicts and list by day"
    - object: schedule
      fields:
        - day
        - instructor_id
      reason: "detect instructor schedule conflicts and list by day"
    - object: observation
      fields:
        - schedule_id
      reason: "load observations for one schedule"
  migration_strategy: |
    Use database-owned versioned init scripts under database/init and seed files
    under database/seed. Each script is idempotent and creates collections,
    validators and indexes if missing. Backend startup never creates indexes or
    seed records. Validation scripts under database/tests check collection and
    index presence before release.
  backup_recovery: |
    For MVP local runtime, document backup and restore commands using mongodump
    and mongorestore against the database service. Production readiness requires
    encrypted backups, restricted access, daily restore test and RPO/RTO values
    before release. Seed data must be synthetic and never include real PII.
  quality_score: 0.91
```
