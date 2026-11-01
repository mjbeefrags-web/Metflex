# CINEFLOW Architecture

**Status:** Draft for review; not approved. **Date:** 2026-10-31.  
The repository has no application code, version pins, Git metadata, or published GitHub issues. These artifacts describe a proposed first-release architecture; they do not assert implementation or test evidence. ADR-0001 through ADR-0003 are proposals, not approved decisions.

## Contents

- [System architecture](system-architecture.md): boundaries, diagrams, media flow, security and operational failure modes. API details live only in the API contract.
- [API contracts](api-contracts.md): proposed endpoint schemas, authorization, errors, pagination, idempotency, and rate limits for journeys A–E.
- [Data model](data-model.md): initial entity relationships, ownership, constraints, and access patterns.
- [ADRs](adr/): decisions proposed for Architect approval.

The diagrams show the PostgreSQL outbox relay path and optional authorized provider path. The data model distinguishes immutable viewing events from resume progress and retains favorites separately from watch-later. It includes per-permission evidence and decision records and at-least-once outbox delivery. The API contract blocks playback without trusted territory and approved post-revocation bounds; those policy inputs remain unapproved.

## Approval gate

Before implementation beyond project setup, the Architect should confirm target clients, authentication model, launch territories, content rights approval process, cloud/region/budget, and operational SLOs. The [API contracts](api-contracts.md) specify proposed schemas and behavior for journeys A–E but remain unapproved. The browser/device and cloud-specific choices remain open. The proposals draw on [research](../research/README.md); see [technical decisions](../research/technical-decisions.md) for source findings and detailed trade-offs.
