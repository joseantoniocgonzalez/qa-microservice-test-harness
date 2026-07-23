# qa-microservice-test-harness

Java microservice with a REST API and a reproducible integration testing setup based on real infrastructure.

The focus of this project is correctness, reproducibility, and end-to-end validation under realistic conditions.

## Project Goals

This repository demonstrates how a microservice can:

- run against a real PostgreSQL database
- manage its schema exclusively via Flyway migrations
- be tested end-to-end (HTTP + database)
- support persistent idempotency
- enforce automated quality gates in CI

## Technology Stack

- Java 21
- Spring Boot
- PostgreSQL
- Flyway (database migrations)
- JPA / Hibernate
- JUnit 5
- RestAssured
- Testcontainers
- Docker / Podman (development environment)
- Maven Wrapper
- Spotless (code formatting)
- JaCoCo (code coverage)
- GitHub Actions (CI)

---

## System Under Test (SUT)

A minimal Tasks REST API with real persistence.

### Health check

`GET /health`

Response:

```json
{ "status": "ok" }
```

---

### Create task (idempotent)

`POST /tasks`

Request body:

```json
{
  "title": "Task description"
}
```

Rules:

- `title` is mandatory
- `title` must not be empty or blank

Optional header:

- `Idempotency-Key`

Behavior:

- same key + same payload -> no duplication, same response returned
- same key + different payload -> HTTP 409 Conflict
- idempotency state is persisted in the database

Examples:

1. First request (creates the task)

```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: demo-key-1" \
  -d '{"title":"Lavar coche"}'
```

Expected: 201 Created (or 200 OK, depending on implementation)

2. Replay (same key + same payload)

```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: demo-key-1" \
  -d '{"title":"Lavar coche"}'
```

Expected: same response as the first call (no duplication)

3. Conflict (same key + different payload)

```bash
curl -i -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: demo-key-1" \
  -d '{"title":"Pintar pared"}'
```

Expected: HTTP 409 Conflict

---

### List tasks

`GET /tasks`

Query parameters:

- `limit` (default 20, min 1, max 100)
- `offset` (default 0)
- `search` (optional, case-insensitive)

---

### Get task by ID

`GET /tasks/{id}`

Responses:

- 200 OK if the task exists
- 404 Not Found otherwise

404 response body:

```json
{ "detail": "Task not found" }
```

---

## Database & Migrations

The service uses a real PostgreSQL database in all environments.

- Schema changes are managed exclusively via Flyway
- Hibernate is configured with `ddl-auto=validate`
- No schema is generated automatically at runtime

---

## Testing Strategy

This project focuses on integration testing over mocked unit tests.

- HTTP API tested end-to-end
- Real PostgreSQL via Testcontainers
- Flyway migrations executed during tests
- Idempotency, pagination, and error cases verified

No external infrastructure is required.

---

## Continuous Integration & Quality Gates

GitHub Actions enforces:

- full test execution
- Spotless formatting checks
- JaCoCo coverage thresholds

The build fails if any quality gate is not met.

---

## Project Status

This project is intentionally feature-complete.

The focus is on:

- correctness
- reproducibility
- realistic testing practices
