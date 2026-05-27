```yaml
ux_contract:
  ux_flows_document: |
    Flujos principales: Gestión de Salas (listar, crear, editar, deshabilitar con confirmación y advertencia de conflictos futuros). Gestión de Grupos de Formación (similar). Gestión de Instructores (incluir tipo permanente/contratista). Gestión de Bloques de Horario (crear con selección de sala, grupo, instructor, fecha/hora; validar fin > inicio y detectar conflictos simultáneos; mostrar advertencia antes de guardar). Cancelación de bloques (cambio de estado, no eliminación). Agregar Observaciones a bloques existentes (texto, autor, fecha, severidad). Flujos de error: mostrar mensajes amigables en español al subir formularios inválidos o conflictos detectados. Estados: carga (skeleton/spinner), vacío (mensaje "No hay elementos"), error (alerta con opción reintentar), éxito (notificación y redirección a lista).
  ui_spec_document: |
    Pantalla principal (Dashboard) con resumen de próximos bloques y enlaces rápidos. Cada entidad (Salas, Grupos, Instructores, Bloques) tiene una página de listado con tabla responsive, búsqueda/filtro por nombre o código, y botón "Agregar". Los formularios de creación/edición se muestran en modal con validación en cliente (campos requeridos, formato horario). Al crear un Bloque, se muestran calendarios/selectores de fecha/hora; si se detecta conflicto, se pinta en rojo el campo y se muestra mensaje específico. Las Observaciones aparecen en un panel expandible dentro del detalle del Bloque. Los deshabilitar/eliminar requiren confirmación. Botón de cancelación de bloque cambia estado a "Cancelado" con fondo gris.
  design_tokens_document: |
    Color primario: #0D6EFD (azul), secundario: #6C757D (gris). Éxito: #198754, advertencia: #FFC107, error: #DC3545, info: #0DCAF0. Fondo claro: #F8F9FA, texto principal: #212529. Tipografía: 'Inter', sans-serif; escala: 0.75rem (caption), 0.875rem (small), 1rem (body), 1.25rem (h5), 1.5rem (h4), 2rem (h3). Espaciado: 4, 8, 12, 16, 24, 32, 48px. Bordes redondeados: 4px en inputs, 8px en tarjetas, 16px en modales. Sombras: suave para tarjetas (0 2px 4px rgba(0,0,0,0.1)), elevada para modales (0 8px 16px rgba(0,0,0,0.2)). Breakpoints: sm 576px, md 768px, lg 992px, xl 1200px.
  screens:
    - Dashboard
    - RoomsList
    - RoomDetail
    - RoomForm
    - TrainingGroupsList
    - TrainingGroupDetail
    - TrainingGroupForm
    - InstructorsList
    - InstructorDetail
    - InstructorForm
    - ScheduleBlocksList
    - ScheduleBlockForm
    - ObservationsPanel
  critical_flows:
    - "Crear bloque con detección de conflictos (validación simultánea en sala, grupo, instructor)"
    - "Deshabilitar entidad con advertencia de bloques futuros afectados"
    - "Agregar observación crítica a bloque existente"
    - "Cancelar bloque con cambio de estado y notificación"
  accessibility_requirements:
    - "Cumplir WCAG 2.1 AA: contraste mínimo 4.5:1 en texto normal"
    - "Navegación completa por teclado (Tab, Enter, Escape en modales)"
    - "Mensajes de error asociados a campos mediante aria-describedby"
    - "Anunciar cambios dinámicos (conflictos, notificaciones) con aria-live polite"
    - "Botones y enlaces con texto descriptivo (no solo iconos)"
    - "Tamaño de objetivo táctil mínimo 44x44 px"
  responsive_targets:
    - "Ancho mínimo soportado: 320px (móvil)"
    - "Tableta: 768px, columnas de tabla se apilan"
    - "Escritorio: >=1024px, layout de dos columnas en formularios"
    - "Sidebar de navegación colapsable en móvil"
    - "Tablas con scroll horizontal en pantallas pequeñas"
  quality_score: 8
```