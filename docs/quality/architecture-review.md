# CINEFLOW Independent Architecture Review

**Status:** Documentation review complete; **not approved for implementation**.  
**Reviewer:** Critical Engineer / QA. **Reviewed:** 2026-10-31 (Africa/Cairo).  
**Issue/revision:** No Git repository or published issue IDs are available.

## Scope and evidence

Reviewed `AGENTS.md`, root `README.md`, engineering protocol, project status/decision register, the research index and technical decisions, planning requirements/milestones/backlog, architecture system/data/API documents and ADR-0001–0003, quality strategy/threat model, and operations infrastructure/delivery/reliability plans.

This review independently compares those repository artifacts with the kickoff requirements. The Director's prior assessment was not used as review evidence. This is a review of documentation only: there is no application implementation, locked dependency set, deployed environment, or executed test evidence. No tests were run and no security, performance, legal, or compatibility certification is implied.

## Review conclusion

The draft is a solid starting architecture: it establishes a modular monolith, separates API and media workers, identifies domain ownership, records a rights gate, describes HLS/CDN delivery, proposes durable job state/outbox, and documents important security and failure concerns. However, several product contracts and cross-document details remain unresolved or inconsistent. M1 is **not ready for approval**, and no feature issue is ready for Builder implementation.

## Findings

### P1 — Reconcile the duplicated API contract descriptions

`docs/architecture/api-contracts.md` now provides detailed proposed schemas, while `docs/architecture/system-architecture.md`, section “API contract policy (proposal),” still presents only an outline table and says schemas/status codes must be specified later. The latter does not point readers to the new contract. This creates two competing sources and could lead implementation to follow the incomplete outline.

**Required:** Make `api-contracts.md` the sole detailed contract source; replace the old outline with a direct link and keep endpoint/schema details in one place. Re-review any differences before approving the contract.

### P1 — Resolve source/delivery rights before selecting the media pipeline

`docs/architecture/system-architecture.md`, “Ingestion / processing,” assumes authorized source or upload is copied/probed and outputs are staged in private object storage. ADR-0002 assumes private-origin/CDN delivery. `docs/planning/requirements.md`, functional requirement 7 and `docs/architecture/api-contracts.md`, “MediaSourceCreate” also allow an authorized third-party source. Authorization to stream does not necessarily authorize downloading, storing, transcoding, creating derivative renditions, or redistributing through CINEFLOW's CDN.

**Required:** For each approved source class, record distinct permissions for acquisition, temporary storage, transcoding/derivative creation, CDN redistribution, territory, term, attribution, and revocation. Document the provider-playback path that avoids CINEFLOW rehosting/transcoding where the grant permits playback only. Do not send content into the current processing pipeline until the grant covers those actions.

### P1 — Define authoritative territory and revocation behavior end to end

`docs/architecture/api-contracts.md`, “Request playback,” correctly says client-supplied territory is only a hint and calls the trustworthy source unresolved. However, the catalog and playback contracts do not specify which trusted edge signal/server-side policy is authoritative, what happens for unknown/ambiguous territory, or how it is tested. The same document proposes a five-minute grant while listing revocation latency as unresolved; `ADR-0002` and the system failure table defer enforcement to a bounded TTL without defining the bound. `docs/planning/requirements.md`, playback acceptance requires rights to be respected at territory and time.

**Required:** Choose the authoritative territory-resolution mechanism and fail-closed behavior; define whether catalog visibility is territory-filtered; set an approved maximum residual-access interval after rights revocation/expiry, covering CDN cache and already issued grants; verify all child manifests, segments and subtitle resources follow the same policy.

### P1 — Complete identity contracts for required account lifecycle

`docs/planning/requirements.md`, functional requirement 1 includes registration, authentication, logout, recovery and account security. `docs/architecture/api-contracts.md`, “Authentication model proposal” explicitly omits password reset/recovery and leaves email verification, MFA, session lifetime and client types unresolved. Those flows affect endpoint contracts, session/CSRF controls, data model and delivery scope.

**Required:** Decide the first supported client/auth model and specify registration verification, recovery/reset, session expiry/revocation, credential-change behavior, rate limits and MFA/admin policy before authentication implementation. Document which are later releases if excluded from MVP; do not infer omission from the current endpoint list.

### P1 — Separate viewing history from continue-watching progress; represent favorites

`docs/planning/requirements.md`, requirement 8 commits watchlists, favorites, progress, continue-watching and viewing history. `docs/architecture/api-contracts.md`, “Watchlists, favorites, history and progress” currently models history as a `ProgressPage` and says favorites may be the same as the watchlist. `docs/architecture/data-model.md` likewise leaves history as progress projection vs append-only viewing events unresolved. These are distinct user-visible capabilities and have different retention/deletion semantics.

**Required:** Define whether history is an event ledger, a completed-title list, or both; distinguish it from resume progress. Decide whether favorites are a separate collection or an explicitly approved watchlist subtype. Define profile ownership, deduplication, retention/deletion, clear-history behavior and API/data contracts.

### P1 — Model the outbox in the schema and state its relay guarantees

`docs/architecture/system-architecture.md`, “Ingestion / processing,” and ADR-0003 require a job and outbox event to commit atomically, followed by a Celery dispatcher. `docs/architecture/data-model.md` ERD/entity list contains `ContentJob` but no outbox record or dispatcher/claim state. The transaction boundary and duplicate dispatch behavior therefore cannot be implemented from the proposed ERD alone.

**Required:** Add the outbox schema or explicitly document its representation, event identity/version, lease/claim and retry model, retention/cleanup, poison-event handling, and consumer idempotency. State delivery as at-least-once with deduplication unless a different guarantee is demonstrated. Align API job state and recovery behavior with that model.

### P1 — Specify the authorization policy behind admin roles

`docs/architecture/api-contracts.md`, “Content administration, rights and ingest” names `content_admin`, `rights_reviewer`, and `publisher`, and proposes separation of duties but leaves role combination unresolved. The system architecture delegates admin policy to domain services, while the data model has no role/permission or service-principal model. Requirements require least privilege and audited rights changes; the threat model calls for MFA and separation of rights approval.

**Required:** Define role-to-action matrix, identity source, privilege assignment/revocation, MFA/re-auth policy, whether submitter may approve/publish, service-worker identity, audit actor representation, and emergency-access review before admin endpoints are implemented.

### P2 — Remove stale duplicate requirements in the system architecture and data model

`docs/architecture/system-architecture.md`, component table still lists ratings in the Library domain. `docs/architecture/data-model.md` includes a `Rating` entity. The planning requirements now explicitly state ratings are not in the kickoff baseline. The system architecture's API outline is also stale relative to `api-contracts.md` (see P1).

**Required:** Remove ratings from the baseline or record an approved scope change; synchronize component, data model and API docs. Avoid implementation work based on features that were not requested.

### P2 — Add recommendation and product-analytics data boundaries

Recommendations and analytics are committed eventual capabilities in `docs/planning/requirements.md`, but the architecture package currently describes telemetry and metrics only. `system-architecture.md` correctly says Prometheus is not a per-user analytics ledger, but there is no proposed event schema, data owner, consent/retention/deletion boundary, recommendation input contract, or rebuild strategy.

**Required:** Before M10 or analytics implementation, document the event/data flow, permitted signals, privacy controls, retention/deletion behavior, and separation of product analytics from operational telemetry. Keep this a later milestone; it does not block initial M2 foundation if no speculative tracking is added.

### P2 — Pin and validate the compatibility baseline before M2

`docs/research/compatibility-matrix.md` explicitly marks the matrix provisional; no package versions, lockfiles, supported browser/device matrix, or FFmpeg build profile exist. The API and infrastructure drafts use current version-sensitive interfaces and codec/CDN behavior. M2's clean build and integration acceptance cannot be verified without these inputs.

**Required:** Before backend/frontend foundation implementation, select supported runtime/package versions and lock them; validate FastAPI/Pydantic/SQLAlchemy/driver/Alembic compatibility, React/router/query/player/browser compatibility, Celery/broker behavior, and FFmpeg output against authorized fixtures. Record exact source versions and target clients.

### P2 — Convert deployment direction into an approved target before production architecture

`docs/operations/infrastructure-plan.md` is appropriately provider-neutral and captures cost inputs, while no cloud, region, workload, SLO, RPO/RTO, data residency or on-call owner is selected. This is acceptable for the current local scaffold but is insufficient to approve production topology or cost/reliability claims.

**Required:** Resolve the provider/region and measurable capacity, cost, availability and recovery objectives before M13. Prove CDN authorization, private-origin access, backup restore, rollback and monitoring against that selected target.

## Readiness gates

Before Architect/Director approve M1, resolve or explicitly assign owners/dates for the P1 items and reconcile duplicated contract text. Before the first backend/API feature, require an approved API contract and exact dependency baseline. Before playback/media work, require source-specific rights covering delivery/processing, trusted territory policy, and tested revocation behavior. Before admin work, require an approved role matrix. Before production deployment, require selected provider/region, SLO/RPO/RTO, cost envelope and recovery evidence.

The first implementation milestone in the proposed roadmap is **M2 Backend and Database Foundation**, but it is **not implementation-ready** until M1 is approved and versions/contracts are pinned. The current safe work is M0/M1 review and decision closure.

## Positive observations

- Product requirements treat authorized streaming as an eventual commitment and title/source availability as rights-dependent.
- Architecture distinguishes technical readiness from rights approval and keeps media delivery outside API workers.
- Threat model and failure plan include cross-account authorization, SSRF, hostile media, CDN token leakage, queue duplicates and rights revocation.
- PostgreSQL, Redis, Celery and CDN roles are labeled as proposals, and the local Compose scaffold is explicitly not production infrastructure.
- Planning retains the kickoff's M0–M13 names and marks local CINE identifiers as unpublished.

No implementation, runtime, security, performance, load, migration, restore or deployment tests were executed in this review.
