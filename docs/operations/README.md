# CINEFLOW Operations

**Status:** Initial plan only; 2026-10-31. No production or application infrastructure is implemented or deployed. A local PostgreSQL/Redis dependency scaffold exists and has only been statically reviewed; it has not been executed or runtime-tested.

This folder is owned by DevOps. The proposed topology follows the draft system architecture and research; those architecture decisions remain subject to Architect and Director review. Read [infrastructure plan](infrastructure-plan.md), [delivery and environment policy](delivery-and-environments.md), and [reliability plan](reliability-plan.md).

## Current state and blockers

- A dependency-only Compose scaffold exists at `infrastructure/compose.yaml`; it has not been executed or validated during this review. No application code, manifests, lockfiles, CI, Git repository, GitHub remote, or deployment target exists.
- Provider, region, traffic, budget, service-level objectives, data residency, retention, and on-call ownership are undecided.
- Draft architecture proposes a single API deployment with isolated Celery workers, PostgreSQL as system of record, Redis as optional auxiliary service, and private object storage behind a CDN. No managed service or vendor has been selected.
- No tests, restore drills, security scans, deployment or monitoring evidence exist.
- PostgreSQL now requires a developer-supplied `POSTGRES_PASSWORD`; Redis remains unauthenticated and loopback-bound. See the static assessment in [infrastructure plan](infrastructure-plan.md). Use the scaffold only for disposable single-developer local development.

## Recommended next action

After Architect approval of the topology and product owners supply region, budget, expected load, and availability targets, create infrastructure issues with acceptance criteria. Extend local Compose only after runtime dependencies and version pins are selected. Treat any production design as blocked until backups/restores, rights-safe media delivery, observability, and rollback are evidenced.
