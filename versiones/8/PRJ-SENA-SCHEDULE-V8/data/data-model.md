```yaml
data_model:
  data_model_document: "This document outlines the logical data model for the SENA Schedule Manager v8, based on the provided PRD and Clean Architecture guidelines. It defines entities, relationships, proposed constraints, PII identification, retention rules, and common query patterns. The model adheres to the specified naming conventions: English, singular, lower_snake_case for all entities and attributes."
  entities:
    - name: instructor
      description: "Represents an instructor within the training center. Can be staff or contractor."
      attributes:
        - name: id
          type: ObjectId
          description: "Unique identifier for the instructor."
          is_primary_key: true
          is_required: true
        - name: name
          type: String
          description: "Full name of the instructor."
          is_required: true
          max_length: 100
        - name: email
          type: String
          description: "Email address of the instructor."
          is_required: true
          is_unique: true
          regex_pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
          max_length: 100
        - name: type
          type: String
          description: "Type of instructor: 'staff' or 'contractor'."
          is_required: true
          allowed_values: ["staff", "contractor"]
        - name: expertise_area
          type: Array
          items_type: String
          description: "List of expertise areas for the instructor."
          is_required: false
          max_items: 10
          max_length_item: 50
    - name: environment
      description: "Represents a learning environment where training can take place (e.g., classroom, lab)."
      attributes:
        - name: id
          type: ObjectId
          description: "Unique identifier for the learning environment."
          is_primary_key: true
          is_required: true
        - name: name
          type: String
          description: "Name or identifier of the environment (e.g., 'Classroom A', 'Lab 2')."
          is_required: true
          is_unique: true
          max_length: 50
        - name: capacity
          type: Number
          description: "Maximum number of individuals the environment can hold."
          is_required: true
          min_value: 1
        - name: type
          type: String
          description: "Type of environment (e.g., 'classroom', 'laboratory', 'virtual')."
          is_required: true
          max_length: 50
        - name: equipment_item
          type: Array
          items_type: String
          description: "List of key equipment items available in the environment."
          is_required: false
          max_items: 20
          max_length_item: 50
    - name: training_group
      description: "Represents a group of trainees associated with a specific training program."
      attributes:
        - name: id
          type: ObjectId
          description: "Unique identifier for the training group."
          is_primary_key: true
          is_required: true
        - name: name
          type: String
          description: "Name or code of the training group."
          is_required: true
          is_unique: true
          max_length: 100
        - name: student_count
          type: Number
          description: "Number of students in the group."
          is_required: true
          min_value: 1
        - name: start_date
          type: Date
          description: "Start date of the training group's program."
          is_required: true
        - name: end_date
          type: Date
          description: "End date of the training group's program."
          is_required: true
          validation_rule: "end_date >= start_date"
    - name: schedule
      description: "Represents a scheduled training session, linking an instructor, group, and environment for a specific time."
      attributes:
        - name: id
          type: ObjectId
          description: "Unique identifier for the schedule entry."
          is_primary_key: true
          is_required: true
        - name: instructor_id
          type: ObjectId
          description: "Reference to the instructor assigned to this schedule."
          is_required: true
          foreign_key_to: instructor
        - name: training_group_id
          type: ObjectId
          description: "Reference to the training group for this schedule."
          is_required: true
          foreign_key_to: training_group
        - name: environment_id
          type: ObjectId
          description: "Reference to the learning environment for this schedule."
          is_required: true
          foreign_key_to: environment
        - name: start_time
          type: Date
          description: "Start timestamp of the scheduled session."
          is_required: true
        - name: end_time
          type: Date
          description: "End timestamp of the scheduled session."
          is_required: true
          validation_rule: "end_time > start_time"
        - name: status
          type: String
          description: "Current status of the schedule (e.g., 'scheduled', 'completed', 'cancelled')."
          is_required: true
          allowed_values: ["scheduled", "completed", "cancelled"]
          default_value: "scheduled"
        - name: description
          type: String
          description: "Brief description of the session content."
          is_required: false
          max_length: 250
    - name: observation
      description: "Represents an operational observation or comment, linked to an instructor or a schedule."
      attributes:
        - name: id
          type: ObjectId
          description: "Unique identifier for the observation."
          is_primary_key: true
          is_required: true
        - name: description
          type: String
          description: "Detailed description of the observation."
          is_required: true
          max_length: 500
        - name: observation_date
          type: Date
          description: "Date and time when the observation was made."
          is_required: true
          default_value: "now()"
        - name: observed_by
          type: String
          description: "Identifier of the user who made the observation (e.g., coordinator ID)."
          is_required: true
          max_length: 100
        - name: reference_type
          type: String
          description: "Indicates whether the observation refers to a 'schedule' or 'instructor'."
          is_required: true
          allowed_values: ["schedule", "instructor"]
        - name: reference_id
          type: ObjectId
          description: "ID of the schedule or instructor this observation refers to."
          is_required: true
          polymorphic_references:
            - entity: schedule
              field: id
            - entity: instructor
              field: id
  relationships:
    - name: instructor_to_schedule
      from_entity: instructor
      from_attribute: id
      to_entity: schedule
      to_attribute: instructor_id
      type: One-to-Many
      description: "An instructor can be assigned to multiple schedules."
    - name: training_group_to_schedule
      from_entity: training_group
      from_attribute: id
      to_entity: schedule
      to_attribute: training_group_id
      type: One-to-Many
      description: "A training group can be associated with multiple schedules."
    - name: environment_to_schedule
      from_entity: environment
      from_attribute: id
      to_entity: schedule
      to_attribute: environment_id
      type: One-to-Many
      description: "An environment can host multiple schedules."
    - name: observation_to_entity
      from_entity: observation
      from_attribute: reference_id
      to_entity: null # Polymorphic relationship, target entity depends on reference_type
      to_attribute: null
      type: Polymorphic
      description: "An observation can be linked to either a schedule or an instructor."
  pii_fields:
    - entity: instructor
      attribute: name
      reason: "Direct identifier of a person."
      privacy_level: "Confidential"
      handling_guidance: "Requires access control. Mask in logs. Encrypt at rest."
    - entity: instructor
      attribute: email
      reason: "Direct contact information, unique identifier."
      privacy_level: "Confidential"
      handling_guidance: "Requires access control. Mask in logs. Encrypt at rest. Used for communication."
    - entity: observation
      attribute: observed_by
      reason: "Identifier of the person making the observation."
      privacy_level: "Internal"
      handling_guidance: "Requires internal access control. Should be system user ID, not full name, if possible."
  retention_rules:
    - entity: schedule
      rule: "Retain for 5 years after end_time, then archive or soft delete. Operational audit trail."
      justification: "Historical record for academic progress and resource utilization."
    - entity: observation
      rule: "Retain for 5 years after observation_date. Associated with schedules/instructors."
      justification: "Compliance and performance review documentation."
    - entity: instructor
      rule: "Retain for 7 years after last active schedule, then anonymize personal data."
      justification: "Legal and re-engagement potential. Anonymize sensitive fields to comply with data minimization."
    - entity: training_group
      rule: "Retain for 7 years after end_date, then archive or soft delete."
      justification: "Historical context for educational programs."
    - entity: environment
      rule: "Retain indefinitely or until facility decommissioning."
      justification: "Static reference data, changes infrequently."
  query_patterns:
    - name: "Get Instructor Schedules"
      description: "Retrieve all schedules for a specific instructor within a date range."
      entities_involved: ["instructor", "schedule"]
      fields_filtered: ["schedule.instructor_id", "schedule.start_time", "schedule.end_time"]
      indexes_proposed:
        - collection: schedule
          fields: ["instructor_id", "start_time", "end_time"]
          type: compound
    - name: "Find Available Environments"
      description: "Find environments available for a given time slot and capacity."
      entities_involved: ["environment", "schedule"]
      fields_filtered: ["schedule.start_time", "schedule.end_time", "environment.capacity"]
      indexes_proposed:
        - collection: schedule
          fields: ["start_time", "end_time"]
          type: compound
        - collection: environment
          fields: ["capacity"]
          type: single
    - name: "Get Group Schedule Overview"
      description: "Retrieve all schedules for a specific training group."
      entities_involved: ["training_group", "schedule"]
      fields_filtered: ["schedule.training_group_id"]
      indexes_proposed:
        - collection: schedule
          fields: ["training_group_id"]
          type: single
    - name: "List Observations for Instructor/Schedule"
      description: "Fetch all observations related to a specific instructor or schedule."
      entities_involved: ["observation"]
      fields_filtered: ["observation.reference_type", "observation.reference_id"]
      indexes_proposed:
        - collection: observation
          fields: ["reference_type", "reference_id"]
          type: compound
  quality_score: 0.95
```