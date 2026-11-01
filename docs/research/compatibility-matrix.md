# Compatibility Matrix

**Status:** Provisional; no versions are pinned and no build exists. Matrix records architectural compatibility, not a tested compatibility claim.

**Version reference points captured 2026-10-31:** React docs reported 19.3 latest; SQLAlchemy docs are 2.0; PostgreSQL current manual is 18; Celery stable docs show 5.6.3; RFC 8216 describes HLS v7 while Apple publishes a newer HLS 2nd Edition. These are documentation references only, not dependency pins. For technologies with rolling docs (including FastAPI, Pydantic, TypeScript, HLS.js, Redis and FFmpeg), exact version selection remains open.

| Component | Intended pairing | Compatibility status | Validation required |
|---|---|---|---|
| React + TypeScript | React DOM, TypeScript compiler, bundler | Supported ecosystem; exact versions unpinned | React/DOM/TypeScript peer and compiler support, browser matrix |
| React Router + TanStack Query | React client | Independent layers; no fundamental incompatibility identified | Pin versions; route loaders vs query cache ownership and SSR mode if used |
| HLS.js | Browser `<video>` + MSE | MSE required for hls.js; Safari/native HLS may use native path | Codec/MSE support, CORS, DRM requirements, target devices and native behavior |
| FastAPI + Pydantic | Python ASGI | FastAPI uses Pydantic; exact major version compatibility must follow FastAPI release support | Pin FastAPI/Pydantic/Python, run OpenAPI and validation integration tests |
| SQLAlchemy async + PostgreSQL | Async dialect (e.g. asyncpg) | Official async support requires compatible dialect | Python/SQLAlchemy/driver/Postgres versions; pool sizing and migration path |
| Alembic + async SQLAlchemy | PostgreSQL schema migration | Supported patterns exist; environment-specific setup required | Fresh DB upgrade, downgrade policy, online migration locking |
| Celery + Redis | Broker/result backend optional | Commonly supported integration; Redis persistence/failure semantics are deployment-specific | Celery/Kombu/Redis versions, broker visibility/ack/retry behavior |
| FFmpeg + HLS output | Worker to object storage | HLS muxer documented; supported codecs depend on build configuration | `ffmpeg -buildconf`, target codec/device checks, golden sample validation |
| Object store + CDN | HLS playlist/segments | Vendor-specific | Signed access inheritance, CORS, MIME, range, cache, token expiry, cost |
| Nginx + FastAPI | Reverse proxy | Supported; forwarded header trust and timeouts need configuration | TLS, buffering, body limits, real IP trust, request IDs |
| Prometheus + Grafana | Operational metrics | Prometheus scrape model documented | Cardinality, retention, alert coverage, scrape authentication |
| OpenTelemetry | Instrumentation/export | Backend-neutral by design | SDK versions, semantic conventions, exporter compatibility, sampling |

**Release gate:** commit lockfiles and deployment image tags, then test a supported-browser matrix, fresh PostgreSQL migration, Celery broker loss/retry behavior, FFmpeg output validation, and CDN authorization/CORS. These are required future checks, not tests performed in this research task.
