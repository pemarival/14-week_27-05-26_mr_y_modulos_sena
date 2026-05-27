```yaml
db_design:
  engine: "MongoDB 7.0"
  physical_objects:
    - name: room
      type: collection
      purpose: "Stores physical or virtual rooms available for scheduling."
      fields:
        - id
        - name
        - capacity
        - location
        - equipment_item
        - is_active
        - created_at
        - updated_at
    - name: training_group
      type: collection
      purpose: "Represents a group of students that receive training together."
      fields:
        - id
        - name
        - code
        - student_count
        - is_active
        - created_at
        - updated_at
    - name: instructor
      type: collection
      purpose: "Represents a permanent or contractor instructor, including PII fields."
      fields:
        - id
        - first_name
        - last_name
        - email
        - phone
        - document_id
        - hire_type
        - expertise_area
        - is_active
        - created_at
        - updated_at
    - name: schedule_block
      type: collection
      purpose: "Represents a scheduled time slot linking room, group, and instructor."
      fields:
        - id
        - room_id
        - training_group_id
        - instructor_id
        - start_time
        - end_time
        - status
        - is_cancelled
        - created_at
        - updated_at
    - name: observation
      type: collection
      purpose: "Attached observation to a schedule block, author PII field."
      fields:
        - id
        - schedule_block_id
        - author
        - text
        - severity
        - created_at
  indexes:
    - object: schedule_block
      fields: [ room_id, start_time, end_time ]
      reason: "Conflict detection: overlapping blocks for a room (is_cancelled=false)."
    - object: schedule_block
      fields: [ training_group_id, start_time, end_time ]
      reason: "Conflict detection for a training group."
    - object: schedule_block
      fields: [ instructor_id, start_time, end_time ]
      reason: "Conflict detection for an instructor."
    - object: observation
      fields: [ schedule_block_id ]
      reason: "Fast retrieval of observations for a schedule block."
    - object: room
      fields: [ is_active ]
      reason: "List active rooms."
    - object: training_group
      fields: [ is_active ]
      reason: "List active training groups."
    - object: instructor
      fields: [ is_active ]
      reason: "List active instructors."
  db_design_document: >
    The database is MongoDB 7.0 running in a replica set for high availability and
    point-in-time recovery. Five collections (room, training_group, instructor,
    schedule_block, observation) store the domain entities with their attributes as
    specified in the data model. References between collections use ObjectId fields
    (room_id, training_group_id, instructor_id, schedule_block_id). All collection
    and field names follow English singular lower_snake_case. PII fields (instructor:
    email, phone, document_id; observation: author) are identified for encryption at
    application level and restricted access. Data retention policy archives records
    5 years after deactivation or cancellation, enforced by periodic TTL and archive
    jobs. The physical design separates database assets under the `database/` root
    with subdirectories `init/` (index creation scripts), `seed/` (sample data), and
    optionally `migrations/` for schema evolution. Domain entities remain pure Go
    structs; mapping to BSON occurs in the repository infrastructure layer.
  migration_strategy: >
    Given MongoDB's schema flexibility, migrations are limited to index changes and
    seed data updates. All index creation scripts reside in `database/init/` and are
    idempotent (using `createIndex` with `background: true`). Seed data is stored in
    `database/seed/` as a JSON file for each entity, loaded via a script that clears
    and re‑inserts. Schema changes (new fields) are handled by application code with
    default values; no explicit ALTER is needed. Backward compatibility for old
    documents is ensured by the repository layer. Scripts are versioned and run
    manually or via CI before deployment. No ORM or migration framework is used;
    migrations are simple shell scripts executed in order.
  backup_recovery: >
    Full backups are taken daily via `mongodump` to a separate volume, retained for
    30 days. Incremental oplog backups are taken every hour (using replica set
    secondary) to enable point‑in‑time recovery within 24 hours. For disaster
    recovery, the latest full backup plus oplog replay restores the cluster to a
    specific second. Archives for deactivated/cancelled records (5‑year retention)
    are stored in a separate cold storage bucket. Backup scripts are located in
    `database/backup/` and are tested monthly via restore drills. Recovery time
    objective (RTO) is 2 hours; recovery point objective (RPO) is 1 hour.
  quality_score: 8.5
```