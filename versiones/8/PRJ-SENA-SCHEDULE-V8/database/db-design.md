```yaml
db_design:
  db_design_document: |
    El diseño de la base de datos física para SENA Schedule Manager v8 se basa en MongoDB, alineándose con los requisitos de la arquitectura. Se utilizarán colecciones para representar cada entidad lógica, con ObjectId como tipo de clave primaria por defecto para mantener la consistencia en el ecosistema MongoDB y facilitar la escalabilidad. Las relaciones se implementarán mediante referencias manuales (IDs) para mantener la normalización necesaria en la capa de persistencia, facilitando la independencia de la lógica de negocio y la flexibilidad para consultas, lo cual es coherente con la Clean Architecture.

    La validación de esquemas JSON se aplicará a nivel de colección para asegurar la integridad y el tipo de datos, replicando las restricciones lógicas y de negocio especificadas en el modelo de datos. Los campos PII como `instructor.name` y `instructor.email` serán identificados y se implementarán controles de seguridad a nivel de aplicación para su gestión, incluyendo el cifrado en reposo. Se priorizarán índices compuestos para patrones de consulta comunes, optimizando el rendimiento de las búsquedas y filtrados de rangos temporales. El objetivo es un balance entre flexibilidad, rendimiento y adherencia a las normas de seguridad y calidad.
  engine: MongoDB
  physical_objects:
    - name: instructor
      purpose: Almacenar la información detallada de cada instructor.
      type: collection
      fields:
        - name: id
          type: ObjectId
          is_primary_key: true
          is_required: true
        - name: name
          type: String
          max_length: 100
          is_required: true
        - name: email
          type: String
          max_length: 100
          is_unique: true
          is_required: true
        - name: type
          type: String
          enum: [staff, contractor]
          is_required: true
        - name: expertise_area
          type: Array<String>
          max_items: 10
    - name: environment
      purpose: Almacenar los detalles de los entornos de aprendizaje disponibles.
      type: collection
      fields:
        - name: id
          type: ObjectId
          is_primary_key: true
          is_required: true
        - name: name
          type: String
          max_length: 50
          is_unique: true
          is_required: true
        - name: capacity
          type: Number
          min_value: 1
          is_required: true
        - name: type
          type: String
          max_length: 50
          is_required: true
        - name: equipment_item
          type: Array<String>
          max_items: 20
    - name: training_group
      purpose: Almacenar la información de los grupos de formación.
      type: collection
      fields:
        - name: id
          type: ObjectId
          is_primary_key: true
          is_required: true
        - name: name
          type: String
          max_length: 100
          is_unique: true
          is_required: true
        - name: student_count
          type: Number
          min_value: 1
          is_required: true
        - name: start_date
          type: Date
          is_required: true
        - name: end_date
          type: Date
          is_required: true
    - name: schedule
      purpose: Registrar las sesiones de formación programadas.
      type: collection
      fields:
        - name: id
          type: ObjectId
          is_primary_key: true
          is_required: true
        - name: instructor_id
          type: ObjectId
          is_required: true
        - name: training_group_id
          type: ObjectId
          is_required: true
        - name: environment_id
          type: ObjectId
          is_required: true
        - name: start_time
          type: Date
          is_required: true
        - name: end_time
          type: Date
          is_required: true
        - name: status
          type: String
          enum: [scheduled, completed, cancelled]
          default: scheduled
          is_required: true
        - name: description
          type: String
          max_length: 250
    - name: observation
      purpose: Almacenar observaciones operacionales relacionadas con instructores o horarios.
      type: collection
      fields:
        - name: id
          type: ObjectId
          is_primary_key: true
          is_required: true
        - name: description
          type: String
          max_length: 500
          is_required: true
        - name: observation_date
          type: Date
          default: now()
          is_required: true
        - name: observed_by
          type: String
          max_length: 100
          is_required: true
        - name: reference_type
          type: String
          enum: [schedule, instructor]
          is_required: true
        - name: reference_id
          type: ObjectId
          is_required: true
  indexes:
    - object: instructor
      fields: [email]
      type: unique
      reason: Garantizar la unicidad del email y optimizar búsquedas por email.
    - object: environment
      fields: [name]
      type: unique
      reason: Garantizar la unicidad del nombre del entorno y optimizar búsquedas por nombre.
    - object: training_group
      fields: [name]
      type: unique
      reason: Garantizar la unicidad del nombre del grupo y optimizar búsquedas por nombre.
    - object: schedule
      fields: [instructor_id, start_time, end_time]
      type: compound
      reason: Optimizar la recuperación de horarios de un instructor en un rango de fechas.
    - object: schedule
      fields: [start_time, end_time]
      type: compound
      reason: Optimizar la búsqueda de entornos disponibles por rangos de tiempo.
    - object: schedule
      fields: [training_group_id]
      type: single
      reason: Optimizar la recuperación de todos los horarios para un grupo de formación específico.
    - object: observation
      fields: [reference_type, reference_id]
      type: compound
      reason: Optimizar la recuperación de observaciones para un instructor o horario específico.
    - object: environment
      fields: [capacity]
      type: single
      reason: Optimizar la búsqueda de entornos por capacidad.
  migration_strategy: |
    La estrategia de migración se centrará en scripts idempotentes de MongoDB alojados en el directorio `database/migrations`. Cada script será versionado y ejecutado secuencialmente para aplicar cambios de esquema o transformaciones de datos. Se utilizará un sistema de control de versiones de migraciones a nivel de aplicación (ej. una colección `db_migrations` en MongoDB) para registrar las migraciones aplicadas y evitar reejecuciones. Los cambios incluirán la creación de nuevas colecciones, adición/modificación de campos, creación de índices y actualización de validadores de esquema JSON. Para PII, cualquier cambio que afecte a `instructor.name` o `instructor.email` requerirá una revisión de seguridad y posiblemente una migración de datos para aplicar cifrado o transformaciones, siguiendo las guías de manejo de PII. La estrategia soportará rollbacks mediante scripts de reversión o la capacidad de aplicar la migración en un entorno de desarrollo/staging antes de producción.
  backup_recovery: |
    La estrategia de backup y recuperación para MongoDB se basará en snapshots periódicos de la base de datos completa. Se implementarán backups diarios incrementales y semanales completos para asegurar puntos de recuperación recientes. Se utilizarán herramientas nativas de MongoDB como `mongodump` y `mongorestore` para la exportación e importación de datos, así como la replicación (replica sets) para alta disponibilidad y redundancia de datos. Los backups se almacenarán en una ubicación segura y geográficamente redundante, con cifrado en reposo para proteger los datos sensibles, incluyendo la información PII. Los planes de recuperación ante desastres (DRP) se probarán anualmente para validar la efectividad de los backups y los procedimientos de restauración, asegurando que el RTO (Recovery Time Objective) y RPO (Recovery Point Objective) sean met. Los logs de operaciones serán monitoreados para detectar anomalías y facilitar la recuperación a un punto en el tiempo específico si es necesario.
  quality_score: 5
```