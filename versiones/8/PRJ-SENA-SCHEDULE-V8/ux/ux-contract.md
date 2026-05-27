```yaml
ux_contract:
  accessibility_requirements:
    - Uso de etiquetas ARIA y roles semánticos para todos los elementos interactivos.
    - Contraste de color mínimo de 4.5:1 para texto e imágenes de texto.
    - Soporte completo de navegación por teclado para todos los flujos.
    - Alternativas de texto descriptivas para todas las imágenes y elementos no textuales.
    - Manejo de estados de foco visible para todos los elementos interactivos.
    - Notificaciones claras para estados de éxito/error/loading usando Live Regions (ARIA-live).
  critical_flows:
    - CRU1: Crear/Editar/Eliminar Entorno de Aprendizaje.
    - CRU2: Crear/Editar/Eliminar Horario (enlazando Instructor, Grupo, Entorno).
    - CRU3: Crear/Editar/Eliminar Grupo de Formación.
    - CRU4: Crear/Editar/Eliminar Instructor (diferenciando staff/contratista).
    - CRU5: Registrar Observación Operacional (enlazando a Horario o Instructor).
    - CRU6: Visualización de Horarios asignados para Instructor (Vista solo lectura).
  design_tokens_document: |
    ### Design Tokens Base (Ejemplos)

    **Colores:**
    - `color-primary-500`: #007BFF (Azul principal)
    - `color-primary-600`: #0056B3 (Azul oscuro para hover/active)
    - `color-secondary-500`: #6C757D (Gris secundario)
    - `color-success-500`: #28A745 (Verde para éxito)
    - `color-danger-500`: #DC3545 (Rojo para error)
    - `color-warning-500`: #FFC107 (Amarillo para advertencia)
    - `color-info-500`: #17A2B8 (Cian para información)
    - `color-text-dark`: #212529 (Texto oscuro)
    - `color-text-light`: #F8F9FA (Texto claro, sobre fondos oscuros)
    - `color-background-light`: #FFFFFF (Fondo principal claro)
    - `color-background-dark`: #F4F6F9 (Fondo secundario claro)
    - `color-border`: #CED4DA (Color de borde)

    **Tipografía:**
    - `font-family-base`: 'Arial', sans-serif
    - `font-size-base`: 16px
    - `font-size-h1`: 32px
    - `font-size-h2`: 24px
    - `font-size-small`: 14px
    - `line-height-base`: 1.5
    - `font-weight-regular`: 400
    - `font-weight-bold`: 700

    **Espaciado:**
    - `space-xs`: 4px
    - `space-sm`: 8px
    - `space-md`: 16px
    - `space-lg`: 24px
    - `space-xl`: 32px

    **Bordes y Sombras:**
    - `border-radius-sm`: 4px
    - `border-radius-md`: 8px
    - `box-shadow-sm`: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075)

    Estos tokens deben ser accesibles a través de variables CSS o un sistema de tematización en React, garantizando consistencia y facilidad de mantenimiento.
  quality_score: 5
  responsive_targets:
    - Desktop (1280px y superior)
    - Tablet (768px - 1279px)
    - Mobile (320px - 767px)
  screens:
    - Nombre: Dashboard (Academic Coordinator)
      Props: Vista general de estadísticas y accesos directos a la gestión de entidades.
    - Nombre: ScheduleList
      Props: Listado paginado de horarios con filtros y acciones de CRUD.
    - Nombre: ScheduleForm
      Props: Formulario para crear/editar un horario, incluyendo selectores de instructor, grupo y entorno.
    - Nombre: InstructorList
      Props: Listado paginado de instructores con filtros por tipo (staff/contratista) y acciones de CRUD.
    - Nombre: InstructorForm
      Props: Formulario para crear/editar un instructor, incluyendo tipo de contrato.
    - Nombre: EnvironmentList
      Props: Listado paginado de entornos con filtros y acciones de CRUD.
    - Nombre: EnvironmentForm
      Props: Formulario para crear/editar un entorno.
    - Nombre: GroupList
      Props: Listado paginado de grupos con filtros y acciones de CRUD.
    - Nombre: GroupForm
      Props: Formulario para crear/editar un grupo.
    - Nombre: ObservationList
      Props: Listado paginado de observaciones con filtros por horario/instructor y acciones de CRUD.
    - Nombre: ObservationForm
      Props: Formulario para registrar una observación, con selectores para asociar a horario o instructor.
    - Nombre: InstructorDashboard (Instructor)
      Props: Vista de solo lectura para instructores con sus horarios asignados y observaciones relacionadas.
    - Nombre: LoginScreen
      Props: Pantalla de autenticación.
    - Nombre: NotFoundScreen
      Props: Pantalla 404.
    - Nombre: UnauthorizedScreen
      Props: Pantalla 403.
  ui_spec_document: |
    ### Especificación de UI General

    **Principios de Diseño:**
    -   **Consistencia:** Elementos de UI, tipografía, colores y espaciado deben ser consistentes en toda la aplicación.
    -   **Claridad:** La información y las acciones deben ser claras y fáciles de entender.
    -   **Eficiencia:** Optimizar flujos de trabajo para coordinadores, minimizando pasos innecesarios.
    -   **Retroalimentación:** Proporcionar retroalimentación clara y oportuna al usuario para cada acción (éxito, error, carga).

    **Componentes Reutilizables (shared/components):**
    -   **Navegación:** Barra lateral para el Coordinador Académico, barra superior para el Instructor.
    -   **Tablas:** Componente de tabla con paginación, ordenamiento, filtrado y acciones por fila.
    -   **Formularios:**
        -   Campos de texto (`InputText`), números (`InputNumber`).
        -   Selectores (`Dropdown` para listas predefinidas, `Autocomplete` para búsqueda en listas grandes).
        -   Selector de fecha/hora (`DateTimePicker`).
        -   Botones (`Button` para acciones primarias/secundarias/peligro).
        -   Validación en tiempo real y al enviar.
    -   **Modales/Alertas:** Modales de confirmación para eliminar, alertas de éxito/error/información (Toast o banners).
    -   **Indicadores de Carga:** Spinners o esqueletos (Skeletons) para indicar carga de datos.
    -   **Estados Vacíos:** Mensajes claros y acciones sugeridas cuando no hay datos.

    **Patrones de Pantalla:**
    -   **Listados CRUD:** Todas las pantallas de listado (Instructors, Schedules, Environments, Groups, Observations) seguirán un patrón similar:
        -   Título de la sección.
        -   Botón "Crear [Entidad]".
        -   Barra de búsqueda global y/o filtros específicos (ej., por tipo de instructor, rango de fechas de horario).
        -   Tabla de datos con columnas relevantes.
        -   Paginación en la parte inferior.
        -   Acciones por fila: "Editar", "Eliminar", "Ver Detalles".
        -   Estados:
            -   **Loading:** Skeleton UI en la tabla.
            -   **Empty:** Mensaje "No hay [entidades] registradas. ¡Crea la primera!" con botón de acción.
            -   **Error:** Mensaje de error visible en la parte superior o debajo de la tabla.
    -   **Formularios CRUD:** Todos los formularios (InstructorForm, ScheduleForm, etc.) seguirán un patrón similar:
        -   Título "Crear [Entidad]" o "Editar [Entidad]".
        -   Campos de entrada organizados lógicamente.
        -   Validación en tiempo real y mensajes de error claros por campo.
        -   Botones "Guardar" y "Cancelar".
        -   Estados:
            -   **Loading (Submitting):** Botón "Guardar" deshabilitado con spinner.
            -   **Error (Submitting):** Mensaje de error general en la parte superior del formulario o Toast.
            -   **Success:** Mensaje de éxito (Toast) y redirección a la vista de listado o detalles.

    **Vistas de solo lectura:**
    -   **InstructorDashboard:** Presenta los horarios asignados al instructor en un calendario o lista, y las observaciones relacionadas de forma tabular. Sin acciones de edición.
  ux_flows_document: |
    ### Flujos de Usuario (MVP)

    **1. Autenticación y Acceso Inicial:**
    -   **Punto de Inicio:** `LoginScreen`.
    -   **Acción:** Usuario ingresa credenciales, click en "Iniciar Sesión".
    -   **Estados:**
        -   `Loading`: Botón "Iniciar Sesión" con spinner.
        -   `Error`: Mensaje "Credenciales inválidas" o "Error de servidor".
        -   `Success`: Redirección a `Dashboard` (Coordinador) o `InstructorDashboard` (Instructor).
    -   **Accesibilidad:** Roles ARIA para formulario, notificaciones de error usando `aria-live`.

    **2. Gestión de Entidades (CRUD - Coordinador Académico):**
    -   **Flujo Genérico (Ej: `Instructor`):**
        1.  **Ver Lista:** Desde `Dashboard`, click en "Instructores" -> `InstructorList`.
        2.  **Crear Nuevo:** En `InstructorList`, click "Crear Instructor" -> `InstructorForm` (vacío).
            -   **Acción:** Llenar formulario, click "Guardar".
            -   **Estados:** `Loading` (spinner en botón), `Error` (validación de campos o error de API), `Success` (Toast, redirección a `InstructorList`).
        3.  **Editar Existente:** En `InstructorList`, click en "Editar" (fila de instructor) -> `InstructorForm` (precargado).
            -   **Acción:** Modificar campos, click "Guardar".
            -   **Estados:** Similar a "Crear".
        4.  **Eliminar:** En `InstructorList`, click en "Eliminar" (fila de instructor).
            -   **Acción:** Modal de confirmación "Está seguro de eliminar a [Nombre Instructor]?". Click "Confirmar".
            -   **Estados:** `Loading` (spinner en modal), `Error` (Toast, error API), `Success` (Toast, actualización de `InstructorList`).
    -   **Entidades Cubiertas:** `Learning Environments`, `Schedules`, `Training Groups`, `Instructors`, `Operational Observations`.
    -   **Accesibilidad:** Navegación por teclado en tablas y formularios, mensajes de confirmación de eliminación con `aria-labelledby`, manejo de foco en modales.

    **3. Gestión de Horarios (Flujo Complejo - Coordinador Académico):**
    -   **Crear Horario:** En `ScheduleForm`, se requiere seleccionar un `Instructor`, `Training Group` y `Learning Environment`.
        -   **Regla de Negocio:** El sistema debe prevenir solapamientos de horarios para el mismo instructor y el mismo entorno.
        -   **UX:** Tras seleccionar fechas y horas, si hay un solapamiento, mostrar un mensaje de error claro al usuario antes de permitir guardar, indicando qué elemento está ocupado.
    -   **Accesibilidad:** Selectores con autocompletado accesibles, notificaciones de conflicto de horario con `aria-live`.

    **4. Registro de Observaciones Operacionales (Coordinador Académico):**
    -   **Acción:** Desde `ObservationList`, click "Registrar Observación" -> `ObservationForm`.
    -   **UX:** El formulario permite asociar la observación a un `Schedule` o un `Instructor` usando selectores. Si se asocia a un horario, los detalles del horario se muestran para contexto.
    -   **Estados:** `Loading`, `Error`, `Success` (Toast, redirección a `ObservationList`).
    -   **Accesibilidad:** Selectores con etiquetas claras, validación visible para campos condicionales.

    **5. Vista de Horarios (Instructor):**
    -   **Punto de Inicio:** Después de iniciar sesión, el Instructor es redirigido a `InstructorDashboard`.
    -   **UX:** Visualización de sus horarios asignados, quizás en un formato de calendario o una lista cronológica. También se muestran las observaciones vinculadas a él. Solo vista de lectura.
    -   **Acciones:** Ninguna acción de modificación.
    -   **Accesibilidad:** Diseño de solo lectura optimizado para lectores de pantalla, tablas accesibles para horarios y observaciones.
```