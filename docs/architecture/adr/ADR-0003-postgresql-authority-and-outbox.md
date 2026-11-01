# ADR-0003: PostgreSQL Authority; Durable Job State and Outbox

- **Status:** Proposed; Architect approval required
- **Date:** 2026-10-31
- **Context:** User, rights, catalog publication, and job state require relational constraints and transactional changes. Background work may be delivered/retried more than once.
- **Decision proposal:** PostgreSQL is authoritative for business state and job lifecycle. Create job plus outbox event atomically; a relay publishes with at-least-once delivery to Celery. Outbox rows use leases for concurrent relay claiming, bounded retries/backoff, deduplication by immutable event ID/business key, and a dead/poison state requiring audited operator replay. Consumers are idempotent and update durable job state. Retention/lease/retry values are proposed in [data-model.md](../data-model.md) and must be tuned; Redis is optional for cache/broker/counters, never the only authoritative record of rights or job completion.
- **Alternatives:** Direct database+broker dual write; treating broker result backend as business state; introduce a separate event platform immediately.
- **Consequences:** Requires relay delivery, lease recovery, dedupe, poison handling, cleanup/reconciliation code, but avoids silent lost enqueue between database commit and broker publish. At-least-once means duplicate messages are expected; exactly-once is not claimed. Eventual job dispatch adds latency. Redis broker durability and outage policy still require deployment design.
- **References:** [Database research](../../research/database-research.md), [backend research](../../research/backend-research.md).
- **Confidence:** High as reliability direction.
- **Issue:** None supplied.
