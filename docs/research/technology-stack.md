# Technology Stack Research

**Status date:** 2026-10-31. Versions below are documented/current reference points, not pin recommendations. Pin exact versions and test the complete matrix at implementation time.

## Version evidence captured

| Technology | Reference documentation observed | Pin status |
|---|---|---|
| React | React's versions page reported 19.3 as latest when checked; use the page for release-time confirmation | Not selected |
| SQLAlchemy | 2.0 documentation stream | Not selected |
| PostgreSQL | Current official manual is PostgreSQL 18 | Not selected |
| Celery | Stable docs identified 5.6.3 | Not selected |
| HLS | RFC 8216 describes protocol version 7 (2017); Apple's HLS 2nd Edition is the newer authoring/specification source | Protocol/profile selection open |
| FastAPI, Pydantic, TypeScript, React Router, TanStack Query, HLS.js, Alembic, Redis, FFmpeg, Docker/Compose, Nginx, Prometheus, Grafana, OpenTelemetry | Current/stable or rolling official documentation was consulted; this research does not freeze specific releases | Not selected |

These reference versions are not a compatible lockfile and are not a statement that the latest releases are mutually compatible. Re-check official compatibility/support notes when manifests are created.

## Recommended starting profile

Use React + TypeScript for the browser client; React Router for route composition; TanStack Query for remote server state; HLS.js only where native HLS is unavailable; FastAPI + Pydantic for the API; SQLAlchemy 2.x async with PostgreSQL; Alembic for schema evolution; Redis for explicitly bounded ephemeral/cache/queue coordination needs; Celery for durable background media workflows; FFmpeg in isolated workers; object storage plus CDN for media delivery. Keep catalog, identity, watch state, licensing, playback authorization, and administration as modules in one backend deployable until measurements justify extraction.

## Decision factors

- Browser support: HLS.js requires Media Source Extensions; native playback varies by browser/device. Always capability-detect native HLS and HLS.js support, and report unsupported playback clearly.
- Async correctness: SQLAlchemy `AsyncSession` is mutable and must not be shared between concurrent tasks. FastAPI async handlers must not perform blocking CPU/transcode work in the event loop.
- Media scale: delivery bytes dominate API bandwidth and cost; API should authorize playback then issue scoped short-lived delivery credentials or a manifest URL. CDN/object storage choice is a deployment decision.
- Search: begin with PostgreSQL full-text/trigram capabilities if product requirements fit; add a dedicated search service only after relevance, language, and scale needs justify its operational burden.
- Workers: media processing is CPU/GPU and I/O intensive. Queue and worker concurrency must be bounded by measured resource use; retries require idempotent jobs and explicit terminal failure handling.
- Security/legal: source metadata must capture rights evidence, territories, term, permitted uses, attribution, and revocation. “Publicly reachable” is not a license.

See the domain documents for pros/cons, limitations, references, and recommendations. No package versions are pinned because the repository has no manifests or lockfiles.
