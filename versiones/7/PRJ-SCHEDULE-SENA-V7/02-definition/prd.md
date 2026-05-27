# Product Requirements Document

## Vision
Provide a reliable, conflict-free scheduling experience for SENA instructors.

## Objectives
- Eliminate scheduling conflicts.
- Reduce time spent on schedule creation.

## Scope
- Instructor management
- Classroom management
- Time slot definition
- Schedule generation and conflict checking

## Requirements
- The system MUST prevent double-booking an instructor.
- The system MUST prevent double-booking a classroom.
- The system MUST allow recording observations for a schedule entry.

## EPCs and HUs
- **EPC-1**: Core Entity Management
  - **FEA-1.1**: Instructor CRUD
    - HU-001: As an admin, I want to create an instructor.
  - **FEA-1.2**: Classroom CRUD
    - HU-002: As an admin, I want to create a classroom.
- **EPC-2**: Scheduling
  - **FEA-2.1**: Schedule Assignment
    - HU-003: As an admin, I want to assign an instructor to a classroom and time slot.
