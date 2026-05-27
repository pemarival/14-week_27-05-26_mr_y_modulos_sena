```yaml
ux_contract:
  ux_flows_document: |
    # UX Flows

    The MVP uses a compact operational dashboard for coordinators. Primary
    navigation exposes Classroom, Student Group, Instructor, Schedule and
    Observation sections. Each catalog section follows the same flow: load list,
    show empty state if no records exist, open a create form, validate required
    fields, submit through the API service, show success feedback and refresh
    the list. Schedule creation is the critical flow: the coordinator selects
    classroom, student group, instructor, day, start time and end time; the UI
    prevents submission when required fields are missing or the end time is not
    later than the start time. Observation creation starts from a selected
    schedule and preserves schedule context.

    Error states are explicit and recoverable: API timeout, validation error,
    empty catalog dependency, and unexpected server error. Loading states are
    inline and do not shift layout. Success states use compact confirmation
    banners. The UI copy may be Spanish for users, while component, route,
    service and test names remain English.
  ui_spec_document: |
    # UI Specification

    The first screen is the actual schedule management workspace, not a landing
    page. Use a fixed top header with product name, a left navigation rail on
    desktop and a compact tab/navigation menu on mobile. The main content area
    contains one active feature view at a time with a dense table/list and an
    adjacent or modal create form depending on viewport width. Cards are used
    only for repeated records or modal surfaces; page sections remain unframed.

    Required screens: dashboard summary, classroom list/create, student group
    list/create, instructor list/create, schedule list/create, observation
    list/create. App composition lives in `src/app/App.jsx`; feature screens
    live under `src/features`; API access lives only in `src/services`; reusable
    controls live in `src/components`; reusable async state hooks live in
    `src/hooks`. `App.jsx` must not call fetch, localStorage or sessionStorage.
  design_tokens_document: |
    # Design Tokens

    Use a quiet operational palette with neutral surfaces, dark readable text,
    one primary action color, one warning color and one danger color. Avoid a
    one-note purple, beige, dark-blue or brown palette. Border radius is 8px or
    less. Typography uses stable sizes, not viewport-scaled text. Tables,
    forms, buttons and status banners must keep fixed responsive constraints so
    labels do not overlap. Primary buttons use icon plus label where the action
    benefits from recognition; destructive actions require clear danger state.
  screens:
    - dashboard_summary
    - classroom_list_create
    - student_group_list_create
    - instructor_list_create
    - schedule_list_create
    - observation_list_create
  critical_flows:
    - "create classroom then use it as schedule dependency"
    - "create student group then use it as schedule dependency"
    - "create instructor with staff or contractor type"
    - "create schedule with valid day and time range"
    - "reject schedule when required fields are missing"
    - "attach observation to a selected schedule"
  accessibility_requirements:
    - "all form controls have labels and validation text"
    - "keyboard navigation reaches lists, forms and submit actions"
    - "focus state is visible on interactive controls"
    - "status messages use text and color, not color alone"
    - "table/list content remains readable at mobile width"
  responsive_targets:
    - "desktop 1440px"
    - "laptop 1024px"
    - "tablet 768px"
    - "mobile 390px"
  quality_score: 0.9
```
