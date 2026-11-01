# Backend Research

## FastAPI and Pydantic

FastAPI builds on Python type hints, dependency injection, OpenAPI generation, and ASGI. Pydantic validates/coerces data at runtime according to model configuration. Keep request/response schemas separate from ORM entities and configure strictness consciously, especially for identifiers, enums, and security-sensitive fields. OpenAPI is the contract artifact; define error formats, pagination, auth, and compatibility policy explicitly.

References: [FastAPI tutorial](https://fastapi.tiangolo.com/tutorial/), [FastAPI dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/), [Pydantic docs](https://docs.pydantic.dev/latest/).

## SQLAlchemy async

SQLAlchemy 2.x async support uses `create_async_engine` and async-compatible dialects (for PostgreSQL commonly `asyncpg`). The official docs explicitly state one `AsyncSession` is not safe for concurrent tasks; use one session per request/task and explicit eager loading or `lazy="raise"` to avoid implicit I/O. Dispose engines at lifecycle shutdown. Avoid running synchronous blocking DB or media work inside async handlers.

Reference: [SQLAlchemy asyncio](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html).

## Dependency injection and modular monolith

Use FastAPI dependencies for request-scoped session, authenticated principal, and policy checks. Keep dependencies at transport boundaries; domain services accept explicit dependencies and return domain results. Separate modules by domain ownership (identity, catalog, rights, playback, user library, analytics) within a single deployment and database initially. Avoid a generic repository abstraction unless it solves a demonstrated persistence concern.

## API versioning and contracts

Start with `/api/v1` if clients must coexist across incompatible releases; additive compatible changes can remain within a version. Use OpenAPI schema review and contract tests in CI. Define cursor or bounded offset pagination, stable sort, filtering semantics, error envelope, rate-limit behavior, and idempotency for job-triggering writes. Do not expose internal ORM fields.

## Background jobs

Celery supports task queues, retries, routing, and workers; task delivery semantics mean tasks may execute more than once in failure scenarios. Design media tasks idempotently, persist workflow state, use bounded retries/backoff, record input/output checksums, and use dead-letter/failed state handling. Never hold a DB transaction open during a long FFmpeg job. Celery documentation describes early acknowledgment defaults; configure acknowledgment/retry behavior based on the desired failure semantics rather than assuming exactly-once execution.

Reference: [Celery tasks 5.6 documentation](https://docs.celeryq.dev/en/stable/userguide/tasks.html).

## Authentication and authorization

Decide first-party session vs delegated OAuth/OIDC. For a browser-only first-party app, server-managed opaque session IDs in `Secure`, `HttpOnly`, appropriately `SameSite` cookies simplify revocation and reduce token exposure; protect state-changing requests against CSRF. If OAuth is used, follow RFC 9700, authorization code + PKCE, exact redirect URI matching, and vetted OIDC libraries. Enforce authorization in backend services on each object/action; never trust client-supplied user IDs or roles.

## Version caveats

No Python version, package versions, database driver, ASGI server, or lockfile exists in this repository. Validate FastAPI/Pydantic/SQLAlchemy/driver compatibility via their supported version tables and a locked environment before implementation.
