# CINEFLOW

CINEFLOW is planned as a professional movie discovery and streaming platform for public-domain, openly licensed, and explicitly authorized content. The product must preserve evidence of distribution/playback rights and enforce territory and term restrictions. Unauthorized scraping, redistribution, and DRM circumvention are out of scope.

## Project state

This repository is in **initialization and architecture preparation**. As of 2026-10-31, there is no application implementation, dependency manifest, test suite, Git repository, or published GitHub issue backlog. Research documents exist under [`docs/research/`](docs/research/README.md). No feature should be treated as complete until its acceptance criteria and verification evidence are recorded in [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md).

## Preferred technology baseline

React, TypeScript, Tailwind CSS, React Router, TanStack Query, HLS.js; Python, FastAPI, Pydantic, SQLAlchemy, Alembic, PostgreSQL, Redis, Celery; FFmpeg, HLS, object storage, CDN; Docker/Compose, Nginx, GitHub Actions, Prometheus, Grafana, OpenTelemetry. This is a preference pending compatibility/version decisions, not a locked implementation stack. See research and decision records.

## Engineering starting point

- Start with a modular monolith; do not create microservices without evidence and an approved ADR.
- Keep media bytes out of API workers; use authorized storage/CDN delivery.
- Treat PostgreSQL as authoritative for product and rights state. Add caches/queues only with defined failure semantics.
- Require rights verification before publishing or authorizing playback.
- Do not begin major feature implementation before architecture, API contracts, acceptance criteria, and infrastructure requirements are reviewed.

## Documentation map

- [`docs/PROJECT_STATUS.md`](docs/PROJECT_STATUS.md): milestone, assignments, blockers, verification state, next priorities.
- [`docs/ENGINEERING_PROTOCOL.md`](docs/ENGINEERING_PROTOCOL.md): roles, coordination, change control, gates, evidence.
- [`docs/PROJECT_DECISIONS.md`](docs/PROJECT_DECISIONS.md): decision register and open decisions.
- [`docs/research/`](docs/research/README.md): technical research and primary references.
- `docs/architecture/`: architecture, contracts, approved ADRs.
- `docs/planning/`: requirements, milestones, local issue backlog.
- `docs/quality/`: quality strategy and independent verification.
- `docs/operations/`: infrastructure and operational plans.
- `infrastructure/compose.yaml`: local Postgres/Redis dependency scaffold only; not production topology.

## Current startup

Read `AGENTS.md`, `docs/ENGINEERING_PROTOCOL.md`, and `docs/PROJECT_STATUS.md` before work. The specialist assignments and readiness are documented in the status file. The workspace has no Git metadata or remote; the connected GitHub lookup found no CINEFLOW or Metflex repository. No issues, branches, or pull requests are linked to this workspace. The local Compose scaffold and research/architecture/planning/quality/operations documents remain unvalidated plans; see their status pages.
