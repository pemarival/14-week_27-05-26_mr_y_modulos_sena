# Data Model

## Entities
- **Instructor**: Represents a teacher.
  - Attributes: `instructor_id` (UUID), `name` (String), `type` (String), `max_hours_per_week` (Int), `email` (String)
- **Classroom**: Represents a physical room.
  - Attributes: `classroom_id` (UUID), `name` (String), `capacity` (Int), `location` (String)
- **TimeSlot**: Represents a slice of time in a week.
  - Attributes: `time_slot_id` (UUID), `day_of_week` (Int 0-6), `start_time` (Time), `end_time` (Time)
- **Schedule**: The assignment mapping.
  - Attributes: `schedule_id` (UUID), `instructor_id` (FK), `classroom_id` (FK), `time_slot_id` (FK), `program_name` (String), `group_code` (String)
- **Observation**: Notes on a schedule.
  - Attributes: `observation_id` (UUID), `schedule_id` (FK), `content` (String), `type` (String), `created_by` (String)

## Relationships
- Instructor 1:N Schedule
- Classroom 1:N Schedule
- TimeSlot 1:N Schedule
- Schedule 1:N Observation
