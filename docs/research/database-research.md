# Database and Data Engineering Research

## PostgreSQL as system of record

Store users, profiles, normalized title metadata, rights grants/evidence, playback authorization state, watchlists, progress, and audit records in PostgreSQL. Use foreign keys, unique constraints, check constraints, and transactions for invariants; migration files are the authoritative schema history. Keep derived analytics and search projections rebuildable.

## Indexing and query plans

PostgreSQL provides B-tree, GIN, GiST, BRIN and other index methods. Select indexes from measured query predicates/order, not every column. Composite indexes depend on leading-column usage; indexes add write and storage cost. Use `EXPLAIN (ANALYZE, BUFFERS)` against representative data and monitor slow queries. Use partial indexes for stable selective predicates when justified.

Reference: [PostgreSQL 18 index types](https://www.postgresql.org/docs/current/indexes-types.html), [indexes](https://www.postgresql.org/docs/current/indexes.html).

## Search

PostgreSQL text search supports lexemes, configurations, ranking, and GIN/GiST indexes; trigram extension supports similarity/fuzzy matching. This can cover initial catalog title/description search without operating another service. It is not automatically equivalent to a tuned multilingual search engine, typo tolerance/relevance controls or large-scale faceting. Validate supported languages, stemming, ranking, and query latency on a representative catalog before committing. Add external search only for evidenced unmet requirements.

References: [PostgreSQL full-text introduction](https://www.postgresql.org/docs/current/textsearch-intro.html), [tables and indexes](https://www.postgresql.org/docs/current/textsearch-tables.html), [pg_trgm](https://www.postgresql.org/docs/current/pgtrgm.html).

## Redis caching and consistency

Use Redis only for data that can tolerate expiry or can be reconstructed: short-lived rate-limit counters, transient coordination, and carefully scoped catalog cache. PostgreSQL remains authoritative. Set TTLs, size bounds, eviction and outage behavior; avoid caching authorization decisions or entitlements without explicit invalidation and bounded staleness. Cache-aside is simple but invalidation races and stampedes require controls. Redis is not a durable analytics ledger.

Reference: [Redis documentation](https://redis.io/docs/latest/).

## Events and analytics

For initial scale, commit business state and an outbox event atomically in PostgreSQL, then publish asynchronously and mark delivery; consumers must be idempotent. Avoid dual-write DB+broker updates without an outbox/reconciliation strategy. Use event schemas with versions and data minimization. Prometheus metrics are operational telemetry, not complete per-user billing/product analytics; use a separate event warehouse/pipeline only when retention, query, privacy, and scale requirements are known.

## Data consistency

Keep watch progress updates monotonic or conflict-aware (client timestamp/version), throttle writes, and define resume position semantics. Model entitlement/rights validity with effective dates and territory, and re-check at playback authorization. Use transactions for watchlist uniqueness and progress updates. GDPR/privacy retention and deletion requirements depend on target jurisdictions and require legal review.
