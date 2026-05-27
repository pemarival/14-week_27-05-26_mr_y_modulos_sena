# Pattern Guide

## Chosen Pattern
Clean Architecture (Domain, Repository, Service, Handler).

## Folder Structure (Level 3)
```
backend/
├── src/
│   ├── domain/
│   ├── repository/
│   ├── service/
│   ├── handler/
```

## Dependency Rules
- `domain` depends on nothing.
- `repository` depends on `domain` and DB driver.
- `service` depends on `domain` and `repository` interfaces.
- `handler` depends on `service`.

## Validation Checklist
- [x] Does domain have framework imports? (Should be NO)
- [x] Do handlers contain business logic? (Should be NO)
- [x] Are filenames snake_case? (Yes)
