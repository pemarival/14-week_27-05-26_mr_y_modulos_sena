```yaml
data_model:
  data_model_document: |
    # Logical Data Model

    The MVP data model uses five aggregate roots: classroom, student_group,
    instructor, schedule and observation. The schedule aggregate references the
    catalog roots by identifier and owns schedule-specific validation such as
    day, start time and end time. Observation belongs to schedule and stores
    follow-up text. Logical names are English, singular and ASCII. No physical
    MongoDB metadata is required in domain objects; physical collection and
    index design is delegated to A13, while backend persistence adapters map
    documents to pure domain entities.

    PII is limited to instructor email and document number. The MVP does not
    include authentication or role management, so coordinator identity is out
    of scope. Retention is conservative: operational schedule records are kept
    while the training period is active, then archived according to academic
    record policy.
  entities:
    - name: classroom
      purpose: "Physical learning environment available for scheduling"
      attributes:
        - name: id
          type: string
          required: true
        - name: code
          type: string
          required: true
        - name: name
          type: string
          required: true
        - name: location
          type: string
          required: true
        - name: capacity
          type: integer
          required: true
    - name: student_group
      purpose: "Training group assigned to schedules"
      attributes:
        - name: id
          type: string
          required: true
        - name: code
          type: string
          required: true
        - name: program_name
          type: string
          required: true
        - name: learner_count
          type: integer
          required: true
    - name: instructor
      purpose: "Instructor assigned to training sessions"
      attributes:
        - name: id
          type: string
          required: true
        - name: document_number
          type: string
          required: true
        - name: full_name
          type: string
          required: true
        - name: email
          type: string
          required: true
        - name: instructor_type
          type: enum
          required: true
          allowed_value: "staff|contractor"
    - name: schedule
      purpose: "Planned training session"
      attributes:
        - name: id
          type: string
          required: true
        - name: classroom_id
          type: string
          required: true
        - name: student_group_id
          type: string
          required: true
        - name: instructor_id
          type: string
          required: true
        - name: day
          type: date
          required: true
        - name: start_time
          type: time
          required: true
        - name: end_time
          type: time
          required: true
        - name: status
          type: enum
          required: true
          allowed_value: "planned|cancelled"
    - name: observation
      purpose: "Follow-up note attached to a schedule"
      attributes:
        - name: id
          type: string
          required: true
        - name: schedule_id
          type: string
          required: true
        - name: note
          type: string
          required: true
        - name: created_at
          type: datetime
          required: true
  relationships:
    - name: schedule_classroom
      from: schedule
      to: classroom
      type: many_to_one
    - name: schedule_student_group
      from: schedule
      to: student_group
      type: many_to_one
    - name: schedule_instructor
      from: schedule
      to: instructor
      type: many_to_one
    - name: observation_schedule
      from: observation
      to: schedule
      type: many_to_one
  pii_fields:
    - entity: instructor
      field: document_number
      classification: personal_identifier
    - entity: instructor
      field: email
      classification: contact
  retention_rules:
    - "schedule and observation records are retained for the active academic period and archived afterward."
    - "instructor contact fields are retained only while needed for schedule coordination."
    - "seed data must be synthetic and must not contain real personal data."
  query_patterns:
    - "list classroom ordered by code"
    - "list student_group ordered by code"
    - "list instructor ordered by full_name"
    - "list schedule by day and classroom_id"
    - "list schedule by day and instructor_id"
    - "list observation by schedule_id"
  quality_score: 0.92
```
