# Technical Decisions and Open Questions

This document contains recommendations for Architect review. They are not approved architectural decisions until the Architect records approval and, where applicable, links a GitHub issue/ADR. At the initial research review (2026-10-31), the repository had no README, ADRs, GitHub issue IDs, or implementation; current coordination/architecture artifacts are maintained in their owning directories.

## Proposed decisions

### R-001 — Start as a modular monolith

- **Problem:** product needs coherent domain boundaries without speculative service operations.
- **Options:** modular monolith; service-per-domain from day one.
- **Evidence:** no measured scale, team topology, or independent deployment need exists.
- **Recommendation:** one API deployment with explicit modules and one PostgreSQL system of record; extract only when a measured bottleneck/ownership need warrants it.
- **Trade-offs:** simplest transactions/operations; scaling/deploying domains independently is limited.
- **Security/performance:** centralized authorization is easier to audit; scale API replicas statelessly and isolate workers separately.
- **Confidence:** high for current unknown scale; revisit with evidence.

### R-002 — PostgreSQL authoritative; Redis optional derived/ephemeral

- **Problem:** preserve transactional user and rights state while enabling low-latency cache/coordination.
- **Recommendation:** PostgreSQL authoritative; Redis only for bounded ephemeral/cached values and queue needs, with defined outage policy.
- **Trade-offs:** simple consistency model; cache invalidation and Redis operations remain costs.
- **Confidence:** high.

### R-003 — Object storage/CDN for media bytes

- **Problem:** long-form media traffic should not traverse API workers.
- **Recommendation:** private object origin with CDN delivery and narrowly scoped short-lived authorization; API authorizes playback and records asset version.
- **Open:** cloud vendor, signing mechanism, origin access control, token TTL, license constraints.
- **Security/performance:** URLs/tokens are bearer secrets; direct delivery reduces API bandwidth and improves cacheability.
- **Confidence:** high at architectural level; vendor settings unverified.

### R-004 — Explicit content rights gate

- **Problem:** prevent unauthorized ingestion or delivery.
- **Recommendation:** publication/playback is blocked unless a rights record covers asset, territory, time, media use, and attribution/restrictions; support revocation/takedown with audit trail.
- **Open:** jurisdictional counsel, proof schema, approval roles, source integrations.
- **Confidence:** high as a product control; legal sufficiency requires counsel.

### R-005 — HLS VOD baseline, validate per device

- **Problem:** adaptive delivery across browsers/devices.
- **Recommendation:** HLS VOD with compatible H.264/AAC rendition baseline, ABR ladder selected from measured source/catalog/device demand, HLS.js for MSE browsers and native capability path where appropriate.
- **Trade-offs:** broad reach and CDN friendliness; multiple renditions increase storage/transcode cost, and native/HLS.js behavior differs.
- **Open:** target devices, codecs, DRM, segment format, bitrates, caption requirements.
- **Confidence:** medium-high.

### R-006 — Background transcode via idempotent worker jobs

- **Problem:** media processing is long-running and resource intensive.
- **Recommendation:** Celery worker pool isolated from API; durable job state in PostgreSQL; idempotent tasks, bounded retry, resource caps, staged output/promotion.
- **Trade-offs:** introduces broker/worker operations; avoids blocking request lifecycle.
- **Open:** GPU/CPU, broker topology, storage, priority and quota policy.
- **Confidence:** high.

### R-007 — Browser session auth default pending product needs

- **Problem:** secure first-party web authentication.
- **Recommendation:** favor opaque server-side sessions in secure HttpOnly cookies for browser-only app; CSRF protection for state-changing calls. If delegated login or native clients are required, select OIDC/OAuth with RFC 9700 practices.
- **Open:** client types, MFA, session duration, identity provider.
- **Confidence:** medium; client/platform requirements unknown.

## Open questions / blockers

1. What are the target launch countries/territories and legal review owner?
2. What cloud/provider/region and monthly egress/transcode budget apply?
3. Which browsers, TVs, phones, and accessibility requirements are acceptance targets?
4. Is account login first-party, OIDC, or both? Are native clients planned?
5. Expected catalog size, concurrent viewers, video hours, and retention?
6. Are uploads by trusted administrators only, and is remote URL ingest prohibited?
7. What product event analytics and retention consent requirements apply?
8. Create/identify issue IDs and ADR process; no issues were available to reference.
