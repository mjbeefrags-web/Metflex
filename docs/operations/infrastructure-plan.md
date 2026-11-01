# Infrastructure Foundation Plan

**Classification:** Recommendations and assumptions, not approved deployment decisions.  
**Owner:** DevOps. **Date:** 2026-10-31. **Issue IDs:** None supplied; local planning IDs will be assigned by Planner.

## Goals and constraints

- Keep first deployment cost-conscious and operable by a small team.
- Follow proposed modular-monolith architecture; scale API and worker processes independently, without starting with microservices or a cluster orchestrator.
- Do not serve media bytes from API workers. Keep object origin private and use CDN delivery with scoped authorization that covers manifests and every child asset.
- Preserve database-authoritative rights, publication, asset-version, and content-job state. Telemetry and cache outages must not corrupt these records.
- Require repeatable builds, secrets separation, health/readiness checks, backups and restore evidence before production.

## Proposed local development topology

Use Docker Compose after application dependencies are pinned. Proposed services:

| Service | Local role | Production note |
|---|---|---|
| `web` | Development frontend server or static build | May be served by edge/CDN; keep media delivery separate |
| `api` | One FastAPI instance | Horizontally scalable stateless replicas behind trusted TLS edge |
| `worker` | Celery process with low bounded concurrency | Separate resource limits/autoscaling policy from API; isolate FFmpeg inputs |
| `postgres` | Disposable local database with named volume | Prefer managed PostgreSQL with encryption, backups/PITR and private network where affordable |
| `redis` | Local broker/cache only if confirmed by approved design | Explicit persistence, ACL, TLS and eviction/failure policy; no authoritative business state |
| `nginx` | Optional local routing parity | Production may use managed edge; if Nginx, harden TLS/proxy headers/limits |

Do not add local Prometheus/Grafana/OTel services by default until the initial API/worker instrumentation and concrete observability need exist. A small Compose profile can add them later. Do not use a local Compose topology as proof of production HA.

All containers should use explicit health checks, non-root users, resource limits, read-only filesystems where practical, minimal pinned images, internal-only database/broker networks, and named volumes for local persistence. Never mount source media into a production API container. Keep production media outside ephemeral container filesystems.

## Proposed production shape (provider-neutral)

1. HTTPS edge/WAF or equivalent receives browser traffic; static web assets are cached separately from API routes.
2. Stateless API instances run in private application networking and reach PostgreSQL, Redis/broker and telemetry endpoints only through scoped identities/network rules.
3. Celery workers run as isolated processes/pools with bounded CPU, memory, disk, concurrency and execution time. Media probing/transcoding does not run in API request workers.
4. Managed PostgreSQL (if selected) is private, encrypted, backed up with PITR, and monitored for connection saturation, storage, replication/backup lag, and failover.
5. Private object storage holds quarantine, staging and immutable validated output. CDN is the only public media path; edge authorization and origin access prevent bypass.
6. An OTel Collector or SDK exporter sends privacy-minimized telemetry to selected backends. Telemetry failure must not block serving or transcode jobs.

**Explicitly not proposed now:** Kubernetes, multi-region active-active, microservices, separate search cluster, separate analytics warehouse. Add these only with a measured requirement, cost/operations assessment, and approved ADR.

## Network and trust boundaries

- Public inbound: HTTPS to edge only; restrict direct API/origin/database/broker access.
- API trusts forwarded host/client/scheme headers only from configured edge addresses. Do not expose debug or profiling endpoints publicly.
- Database and Redis accept traffic only from named workload identities/network segments; use TLS where supported and least-privilege credentials.
- Worker egress is deny-by-default except approved package/runtime endpoints and explicitly configured media sources/storage. Arbitrary URL fetch remains disabled unless SSRF design is approved.
- Object origin denies anonymous reads and direct bucket access. CDN has scoped origin identity; user grants are short-lived, asset/path scoped and redacted from logs.
- Development/staging/production have separate accounts, projects, buckets, database instances, credentials and encryption keys.

## Resource sizing and scaling signals

Do not set instance sizes without a workload model and benchmark. Establish a baseline from API concurrency, DB pool use, queue age, CPU-seconds per media minute per rendition, storage growth, video minutes delivered, and regional CDN egress. Scale API from request latency/CPU/queueing, workers from queue age and concurrency budgets, and database vertically/with managed read replicas only after query/connection evidence. Each worker class should have an explicit concurrency cap and queue routing for CPU-heavy jobs.

## Cost model inputs

Obtain provider quotes only after region and workload are known. Model monthly totals for:

- Database instance, storage, IOPS, backups/PITR, cross-zone transfer and connection proxy if needed.
- Redis/broker instance, persistence/replication, network and monitoring.
- API/worker compute (CPU/GPU), autoscaling headroom, idle staging/test resources.
- Object storage GB-month, PUT/LIST/GET requests, lifecycle/versioning, replication and retrieval fees.
- CDN egress by territory/bitrate/viewing hours, request count, cache hit ratio, origin shield and invalidations.
- Logs/traces/metrics ingestion, retention, cardinality, dashboards/alerts and export costs.
- Domain, TLS/WAF, secrets/KMS, vulnerability scanning, backup retention, and operational support.

Use a cost calculator based on viewing hours × average delivered bitrate for bandwidth, plus retained renditions and source upload volumes for storage. Include a monthly budget alert and isolate dev/staging spend. Costs are provider and region specific; no pricing is asserted in this document.

## Dependencies on architecture and product

- Architect approval of boundaries and deployment context; ADR-0001 to ADR-0003 are proposed, not approved.
- Approved Redis topology: broker vs cache, persistence, failure behavior, and whether separation is required.
- CDN/storage vendor and authorization model; prove signed access protects child manifests, segments and subtitle resources.
- Supported media codecs, source handling, ingestion constraints, and legal territories.
- Expected usage, target availability/latency, recovery objectives, privacy/retention, and operations/on-call capacity.

## Existing scaffold assessment (inventory only)

The repository contains `infrastructure/compose.yaml` and `infrastructure/README.md`. It currently defines only PostgreSQL and Redis; it does not run the web app, API, Celery worker, Nginx, CDN, media store, or telemetry. No Docker command or deployment test was run during this assessment.

| Observed setting | Assessment |
|---|---|
| PostgreSQL image `postgres:16.4-alpine`; Redis image `redis:7.4.1-alpine` | Version tags are explicit but not immutable digests; image rebuilds can resolve differently over time. Verify supported patch updates and pinning policy before shared/production use. |
| Host ports default to `127.0.0.1` (5432/6379) | Limits normal remote exposure, but local-host processes/users can still connect. Do not treat loopback publication as authentication or a shared-host security boundary. |
| PostgreSQL requires a non-empty `POSTGRES_PASSWORD` from the developer environment | No committed password fallback is present. Supply a unique local secret in the shell; do not store it in tracked files. |
| Redis starts with AOF enabled and no password/ACL in the Compose command | Acceptable only as a loopback-bound disposable local dependency. Persistence does not establish backup/recovery guarantees. Do not expose it to a network or reuse as a shared broker without approved auth, network, and durability configuration. |
| Named volumes persist service data; `down -v` removes volumes | Local persistence only, not backup. Document the destructive nature of volume removal before use. |
| One `internal: true` network; `no-new-privileges` security option | Basic containment intent; no production hardening, service identities, capability drop, read-only filesystem, CPU/memory limits, TLS, or network policy is represented. |
| Health checks exist | These check only PostgreSQL readiness and Redis ping; they do not validate CINEFLOW migrations, credentials from the API, job flow, backups, or application readiness. |

**Safety conclusion:** configuration is narrowly scoped as a local dependency scaffold and should not be used for production or untrusted multi-user hosts. Redis remains unauthenticated and is safe only within the current loopback-only local development scope. This review is static only; safe execution has not been verified. Compose runtime changes require a tracked DevOps issue, review of the eventual application contract, and acceptance criteria; the password fallback removal was made as an explicitly authorized local security correction.
